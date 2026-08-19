# Rate Limits

Buffer runs three stacked windows on every endpoint. Hitting any one returns HTTP 429 with `Retry-After`.

## Windows

| Name  | Window     | Purpose          |
| ----- | ---------- | ---------------- |
| `15m` | 15 minutes | burst protection |
| `24h` | 24 hours   | daily fairness   |
| `30d` | 30 days    | quota            |

Limits per API key; not per-channel and not per-org. Concurrent API keys have independent budgets.

## Headers

CLI parses both forms:

- `RateLimit` + `RateLimit-Policy` (RFC draft-8) — preferred, comma-separated, one entry per window.
- Legacy `X-RateLimit-Limit/Remaining/Reset` — single window only; fallback.

## Where to read remaining quota

Today rate-limit info is reported on **stderr only**. Stdout JSON does not carry it.

- Default: a one-line warning per policy that crosses the low-remaining threshold (`remaining / limit < 0.10`). Format: `Warning: Rate limit [15m]: 8/100 remaining, resets in 4m 12s`.
- `--verbose`: stderr also receives a one-line summary for every policy (low or not).
- `--quiet`: suppresses both.

Agents that need to inspect quota programmatically should capture stderr separately and grep for `Rate limit`. A structured `meta.rateLimit` field on stdout JSON is planned but not shipped yet.

## On 429

Exit code `3`, error message includes which window tripped and the parsed `Retry-After` (seconds). Example:

```
Rate limit exceeded (15m window). Retry after 4m 12s.
```

The CLI does **not** auto-retry. The agent owns the retry policy:

1. Read `Retry-After` from the message or response headers (when scripting `fetch` directly).
2. Sleep that many seconds.
3. Retry once. If you hit 429 again, double the sleep and retry once more. Then surface the failure.

## Batching strategy

Cheapest call > expensive loop. Prefer:

| Instead of                            | Use                                                           |
| ------------------------------------- | ------------------------------------------------------------- |
| Loop `channels get` for every id      | `channels list --organization-id <id>` once, filter with `jq` |
| Loop `dailyPostingLimits` per channel | One call with `--channel-ids id1,id2,id3,...`                 |
| Loop `posts get` for many ids         | `posts list --organization-id <id>`, paginate, filter         |

## Concurrency

Serial calls are preferred. Parallelism multiplies 429 risk because all in-flight requests count against the same window. If you must parallelise, cap at 2-3 concurrent and pass `--verbose` to surface every window's remaining count between batches.

## What does NOT count

- `--dry-run` invocations (no API call made).
- `buffer schema *`, `buffer config *`, `buffer doctor`, `buffer context`, `buffer --help`, `buffer --version` — all local.
- Update-check pings hit a separate registry, not the API.
