<!-- Illustrative example of a per-branch journal entry
     (JOURNAL/feature/<branch-slug>/03_*.md). Not real project state;
     the domain is invented to show the format. -->

# 03 — request cache keyed on normalized URL

Follow-up to 01 (cache skeleton) and the plan in 02. Implements the
on-disk response cache and wires it into the client.

## What changed

- `cache/store.*` — disk-backed key→response store, keyed on the
  *normalized* request URL (lowercased host, sorted query params).
- `client/fetch.*` — checks the store before hitting the network and
  honours the upstream `Expires` header for the TTL.

## Why / alternatives considered

Keying on the raw URL double-fetched `?b=2&a=1` and `?a=1&b=2`.
Normalizing first collapses them. Considered hashing the parsed
query map instead — rejected because sorting the params keeps the
on-disk key human-readable, which is what made the gotcha below
obvious during debugging.

## Gotcha

The upstream sends `Expires` in RFC 1123 with a trailing `GMT`. The
first parser silently returned epoch 0 on parse failure, so every
response looked "expired in 1970" and was never served from cache.
Parse failures now error loudly instead of defaulting to zero —
**re-check this if the date handling is ever touched.**

## Tests

- `cache::store::roundtrip`, `cache::store::expiry`,
  `client::fetch::serves_from_cache` — 11 passed; 0 failed.

## Open threads

- No cache eviction yet; the store grows without bound. Plan: a
  `purge_expired` pass on startup. Deferred — not load-bearing for
  this feature.
- Normalization doesn't handle percent-encoding case (`%2F` vs
  `%2f`). Rare; flagged.
