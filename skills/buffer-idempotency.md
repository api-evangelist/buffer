# Idempotency

Whether a command is safe to retry. There is no idempotency-key mechanism in the API today; retry safety is a function of the operation alone.

## Reference table

| Command                                     | Idempotent        | On retry                                              |
| ------------------------------------------- | ----------------- | ----------------------------------------------------- |
| `account`                                   | yes               | same response                                         |
| `channels list`                             | yes               | same response (paginated)                             |
| `channels get`                              | yes               | same response                                         |
| `posts list`                                | yes               | same response (paginated)                             |
| `posts get`                                 | yes               | same response                                         |
| `dailyPostingLimits list`                   | yes               | same response                                         |
| `schema *`, `config *`, `doctor`, `context` | yes               | local only                                            |
| **`posts create`**                          | **NO**            | duplicate post created                                |
| **`ideas create`**                          | **NO**            | duplicate idea created                                |
| `posts delete`                              | yes (effectively) | second call returns exit 3, "not found"; data is gone |

## Retry rules

When a command exits non-zero, decide by exit code AND idempotency:

| Exit             | Reads                  | Idempotent mutations   | Non-idempotent mutations |
| ---------------- | ---------------------- | ---------------------- | ------------------------ |
| 1 (general)      | retry once             | retry once             | inspect first            |
| 2 (usage)        | never retry            | never retry            | never retry              |
| 3 (API, 5xx-ish) | retry with backoff     | retry with backoff     | inspect first            |
| 3 (API, 429)     | honor `Retry-After`    | honor `Retry-After`    | honor `Retry-After`      |
| 4 (auth)         | refresh API key, retry | refresh API key, retry | refresh API key, retry   |

"Inspect first" means: list the resource (`posts list` filtered by `channelId` and recent `createdAt`) to detect whether the previous attempt actually landed before retrying.

For the full 429 backoff policy (sleep, double, retry once more) see [`rate-limits.md#on-429`](./rate-limits.md#on-429).

## Practical agent loop for `posts create`

```bash
attempt() { buffer posts create --json "$payload" --output json; }
result=$(attempt) || code=$?

if [ "${code:-0}" -eq 3 ]; then
  # Either request failed mid-flight, or the server processed it and the response was lost.
  # Search recent posts for a match before retrying.
  recent=$(buffer posts list --organization-id "$orgId" --output json \
    | jq --arg ch "$channelId" --arg t "$text" \
        '.items[] | select(.channel.id == $ch and .text == $t) | .id')
  if [ -n "$recent" ]; then
    echo "Already exists as $recent — not retrying"
  else
    sleep 5 && attempt
  fi
fi
```

## Notes

- `--save-to-draft` produces a draft Post — also non-idempotent. Drafts can be deleted cheaply with `posts delete`, so the cost of a duplicate is low. Use drafts for any pipeline you might re-run.
- `posts delete` failing with exit 3 + "not found" is the success signal on retry. Map it to "already deleted" rather than treating as error.
- Read-only commands have no side effects — retry freely, but they still count against rate limits.
