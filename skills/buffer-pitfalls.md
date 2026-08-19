# Pitfalls

Things the schema accepts that still produce wrong behavior. Skim this file before composing any payload.

## Scheduling

- **`mode: addToQueue` is queued, not immediate.** Use `mode: shareNow` to publish now. `shareNext` jumps to the front of the queue. `customScheduled` requires `dueAt`.
- **`schedulingType: notification` does not auto-publish.** It pings the mobile app for a human to publish. Use `schedulingType: automatic` for hands-off publishing.
- **`addToQueue` on a channel with no schedule** silently lands in an empty slot. Inspect the channel with `buffer channels get --id <id>` first.
- **All times are ISO-8601 with offset.** `dueAt` must include a timezone (e.g. `-05:00`). Compute the offset from `buffer config get timezone` (persisted by `buffer init`). Never assume UTC.

## IDs

- **Never guess channel IDs.** Always fetch with `buffer channels list`. IDs look pseudo-random; the API will accept malformed IDs and reject at execute time with vague errors.
- **IDs are not portable across organizations.** A `channelId` from org A cannot be used while authenticated against org B.

## Input

- **`--json` overrides flags entirely.** When both are supplied, every flag is dropped. Pick one form per invocation.
- **Nested objects need `--json`.** Per-service `metadata.*` and `assets.*` cannot be set via flags. Use the JSON path or check `aliases` in `--help`.
- **Empty `text` with no assets is rejected** by the API for most channels even though the schema marks `text` optional.
- **Control characters (U+0000–U+001F except whitespace) are rejected** before the request leaves the CLI. Strip them in the agent.

## Per-service minimum payloads

| Service                                | Required besides `text`                                                                                |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| Twitter / Mastodon / Threads / Bluesky | text only                                                                                              |
| Facebook                               | text or `assets`                                                                                       |
| Instagram                              | image or video asset; `metadata.instagram.type` + `metadata.instagram.shouldShareToFeed`               |
| TikTok                                 | image or video asset                                                                                   |
| Pinterest                              | image asset; `metadata.pinterest.boardServiceId` (from `channels get` → `metadata.boards[].serviceId`) |
| YouTube                                | video asset; `metadata.youtube.title`; `metadata.youtube.categoryId`                                   |
| Google Business                        | `metadata.google.type` (and `detailsOffer` / `detailsEvent` / `detailsWhatsNew` for that type)         |
| LinkedIn                               | text or `assets`; documents need `metadata.linkedin.linkAttachment` semantics                          |

For exact per-service shapes including required nested fields, run:

```bash
buffer schema describe posts create --output json | jq '.jsonInputSchema.properties.metadata'
```

## Threads (multi-post chains)

Platforms with `metadata.<service>.thread`: outer `text` MUST equal the first thread item's `text`. Backend validation rejects mismatches; the schema does not. Always provide both:

```json
{
  "text": "first item",
  "metadata": {
    "twitter": { "thread": [{ "text": "first item" }, { "text": "second" }] }
  }
}
```

## Reading responses

- **`createPost` returns a union.** Inspect `__typename` or use `buffer schema describe posts create` → `output.successTypes`. A "successful" response can still carry validation messages — check fields, don't trust the exit code alone.
- **Pagination cursors are opaque.** Don't parse or compare them. Loop on `pageInfo.hasNextPage`.
- For `--fields` shell-quoting and exit-2 behaviour see [`getting-started.md`](./getting-started.md#trim-noise).

## Cross-org operations

- `dailyPostingLimits list` requires every `channelIds` entry to share one organization.
- `posts list` is per-org. To search across orgs, loop one `posts list` per org.

## Plan limits

Plan limits (paid tiers, channel counts, daily posting cap) are enforced server-side. The CLI does not pre-check. A `posts create` may succeed schema validation and still fail with exit 3 + `OVER_LIMIT` style message. Read `dailyPostingLimits` ahead of bulk runs to detect the cap.

## Dates

- `posts list` filters `dueAt` and `createdAt` accept `{start, end}` as ISO strings. Either bound is optional.
- `dailyPostingLimits` `date` is a calendar date; time component is ignored, treated as the channel's local day.
