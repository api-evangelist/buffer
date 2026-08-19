# Getting Started

Read this once before any other context file. Everything below is invariant across commands.

## Output

- Stdout: JSON only when `--output json` (use this in agents). Stderr: structured errors as JSON `{error, code, message}`, plus spinners and progress.
- `--quiet` suppresses non-essential stderr — update notices, spinners, rate-limit warnings — without touching stdout.

## Exit codes

| Code | Meaning                                          |
| ---- | ------------------------------------------------ |
| 0    | success                                          |
| 1    | general error                                    |
| 2    | usage error (bad flags, validation)              |
| 3    | API error (server returned error, including 429) |
| 4    | auth error (missing/invalid API key)             |
| 130  | interrupted by SIGINT (Ctrl-C)                   |
| 143  | terminated by SIGTERM                            |

EPIPE on stdout (consumer closed the pipe, e.g. `... | head`) exits 0 silently — no error envelope.

## Discovery

Don't guess command shapes. Ask the CLI:

```bash
buffer schema list                            # all commands + descriptions
buffer schema describe posts create           # input flags, types, enums, output shape, jsonInputSchema
buffer schema describe posts create --output json | jq '.jsonInputSchema'
```

`schema describe` returns the full JSON Schema for `--json` payloads, including per-service nested metadata. Always consult before constructing complex inputs.

## Input precedence

Command flag → env var → repo config (`.buffer/config.json`) → global config → built-in default.

`--json` overrides flags entirely when both are passed. Never mix the two on one invocation.

`buffer config get <key>` and `buffer config get --all` surface the resolved layer per entry: `source` is `env` | `repo` | `global` | `default`, `path` carries the file path for file-backed sources, and `envVar` names the variable (e.g. `BUFFER_API_KEY`) when `source` is `env`. Use this to confirm an env override won — or didn't.

## Validate before sending

Every mutation supports `--dry-run`. Returns the validated payload with no API call. Always dry-run a generated payload first; cheap and exits 0 only if input parses.

```bash
buffer posts create --json "$payload" --dry-run
```

## Trim noise

`--fields` is rendered into the GraphQL request itself, so the API skips any field you don't ask for. Prefer it over piping to `jq`.

```bash
buffer posts get --id <id> --fields id,status,dueAt          # top-level scalars
buffer posts get --id <id> --fields id,channel.name          # dotted paths
buffer posts list --fields items.id,pageInfo.endCursor       # connection items
buffer posts list --fields 'items.{id,text,status}'          # brace expansion (single-quote!)
buffer posts get --id <id> --fields all                      # full response
```

Each command has a curated default set; omit `--fields` to use it. Run `buffer schema describe <command>` to see `defaultFields` and the full `selectableFields`. Unknown paths exit 2 (usage error). Use `jq` only for reshaping or aggregation, not for trimming fields.

> Single-quote any value containing `{…}` — bash/zsh expand braces before argv reaches the CLI.

## Glossary

- **Organization** — workspace containing channels, posts, members. Account may have several; pick one explicitly.
- **Channel** — connected social account (e.g. one Instagram profile, one LinkedIn page).
- **Post** — content scheduled or published to one channel.
- **Posting Schedule** — per-channel time slots for `addToQueue` mode. Inspect with `buffer channels get`.
- **Idea** — content draft, not bound to a channel. Promote to a Post by passing `--idea-id` to `posts create`.

## Metrics

Performance numbers (reactions, comments, impressions, reach, etc.) are available per-post via `Post.metrics` (opt in with `--fields`) and in bulk via `buffer aggregatedPostMetrics`. The schema is `@experimental` — shape may change. See [`post-metrics.md`](./post-metrics.md) for normalized metric names, freshness semantics, and example aggregate/per-post workflows.

## Timezones

`buffer init` fetches the account `timezone` (e.g. `America/Panama`) and stores it in global config — read it with `buffer config get timezone` instead of calling `buffer account` for every command. Build `dueAt` in that zone, never assume UTC. Example: user says "5 PM" + zone `America/Panama` → `2026-05-05T17:00:00-05:00`.

Override the persisted zone any time with `buffer config set timezone <IANA-zone>`. If `buffer config get timezone` returns no value (older configs predate this), run `buffer init` again or fall back to `buffer account --fields timezone --output json`.
