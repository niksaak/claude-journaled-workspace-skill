<!-- Illustrative example of a toplevel entry written by consolidating
     a finished branch (JOURNAL/toplevel/07_*.md). Not real project
     state. Note how it distills the per-branch entries rather than
     concatenating them, and carries their open threads forward. The
     `> Origin:` marker under the heading records provenance — here
     `branch <name>`. Trunk-direct entries use `> Origin: trunk-direct`
     and later *related* trunk work updates them in place rather than
     adding a new entry. -->

# 07 — response caching
> Origin: branch feature/response-cache

Consolidates the `feature/response-cache` branch (per-branch entries
00–04, now distilled and removed). Adds an on-disk response cache to
the client so repeated runs inside the upstream cache window avoid
the network.

## What shipped

- Disk-backed response store (`cache/store.*`) keyed on the
  normalized request URL (lowercased host, sorted query params).
- The client checks the store before fetching and honours the
  upstream `Expires` header for the TTL (`client/fetch.*`).
- First run populates the store; subsequent runs inside the cache
  window read entirely from disk.

## Key decisions

- **Normalize the URL before keying** rather than hashing the parsed
  request, so equivalent query orderings collapse to one entry and
  keys stay human-readable for debugging.
- **Disk, not in-memory** — the tool runs as one-shot invocations,
  so the cache must survive process exit to be worth anything.

## Gotcha worth keeping

Upstream `Expires` is RFC 1123 with a trailing `GMT`. A silent
parse-failure default (epoch 0) made every response look expired and
defeated the cache entirely; parse failures now error loudly.
Re-check this if the date handling is ever touched.

## Open threads (carried forward)

- **No cache eviction.** The store grows without bound; a
  `purge_expired` pass on startup is the obvious next step. (From
  branch entry 03.)
- **Percent-encoding case** in URL normalization (`%2F` vs `%2f`) is
  unhandled. Rare. (From branch entry 03.)
