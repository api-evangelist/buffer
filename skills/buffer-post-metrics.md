# Post Metrics

Post-level performance numbers (reactions, comments, impressions, reach, etc.) — surfaced on individual posts via `Post.metrics` and in bulk via `buffer aggregatedPostMetrics`. Schema is `@experimental`: the shape can change without notice.

## Two access patterns

```bash
# Per-post: opt in via --fields. Default `posts list` / `posts get` responses
# stay lean and do NOT include metrics; you must ask for them.
buffer posts get --id "$postId" \
  --fields id,metrics.type,metrics.value,metricsUpdatedAt

buffer posts list \
  --fields 'items.{id,metrics.{type,value,unit},metricsUpdatedAt},pageInfo'

# Aggregate: one query rolls up every post in the window, no pagination.
buffer aggregatedPostMetrics --json '{
  "organizationId": "'"$orgId"'",
  "startDateTime": "2026-01-01T00:00:00Z",
  "endDateTime":   "2026-03-31T00:00:00Z"
}' --output json
```

Default `aggregatedPostMetrics` fields: `metrics.{type,name,value,unit}` plus `metricsUpdatedAt`. Add `metrics.description` with `--fields` when you need the human-readable blurb.

## Normalized metric names

Cross-network metrics are unified under a single name (e.g. Twitter `retweets`, Mastodon `reblogs`, and Threads `reposts` all surface as `reposts`). Network-specific metrics keep their network's vocabulary (`saves` on Instagram/Pinterest, `quotes` on Threads, `viewers`/`totalTimeWatched` on LinkedIn).

Active values you can expect on a `PostMetric.type`: `reactions`, `comments`, `shares`, `reposts`, `reach`, `impressions`, `views`, `clicks`, `engagementRate`, `saves`, `follows`, `quotes`, `viewers`, `totalTimeWatched`, `likes`, `postCount`. Deprecated values (`replies`, `favorites`, `reblogs`, `retweets`, `repins`, `link_clicks`, `other`) are kept for back-compat until 2026-12-01 — never request them in new code. Each value carries a docstring in the GraphQL schema describing its source network; read them via your GraphQL client or the public schema (`PostMetricType` enum).

`postCount` only appears in aggregate responses (as a `PostMetric` entry with `type: postCount`), never per-post. `engagementRate` is the only value with `unit: percentage` — everything else is `unit: count`.

## Cross-network aggregation rule

`aggregatedPostMetrics` always returns the baseline trio (`postCount`, `reactions`, `comments`). Beyond that, a metric is included only when **every** channel in the filter set supports it. So:

- No `channelIds` filter, mixed networks → just the baseline trio.
- `channelIds` filter scoped to a single LinkedIn channel → baseline plus `impressions`, `reach`, `engagementRate`, `viewers`, `totalTimeWatched`.
- `channelIds` filter scoped to Instagram + Threads → baseline plus whatever both networks support.

Filter narrower to get richer numbers.

## Data freshness

Metrics ingest once per day, so `metricsUpdatedAt` can lag the network by up to ~24h. Always read it before reporting numbers as current:

```bash
result=$(buffer aggregatedPostMetrics --json "$payload" --output json)
echo "$result" | jq '.metrics, .metricsUpdatedAt'
```

If `metricsUpdatedAt` is older than ~36h, the ingestion likely missed a cycle — report the staleness rather than presenting the number as live. The field is null when no posts matched the filter.

## Workflows

### Quarterly aggregate for a single channel

```bash
buffer aggregatedPostMetrics --json '{
  "organizationId": "'"$orgId"'",
  "startDateTime":  "2026-01-01T00:00:00Z",
  "endDateTime":    "2026-03-31T00:00:00Z",
  "channelIds":     ["'"$channelId"'"]
}' --output json | jq '.metrics'
```

Date range is server-capped at 365 days — split larger windows client-side.

### Per-post pull with metrics

```bash
buffer posts get --id "$postId" \
  --fields 'id,text,sentAt,metrics.{type,value,unit},metricsUpdatedAt' \
  --output json
```

Use `posts list` with the same `--fields` shape to fan out across the feed and compute your own ranking client-side. For cross-post ranking that's already sorted, prefer `topPosts`.

### Filtering by tags

`aggregatedPostMetrics` takes a `tags` filter via `TagComparator` — useful for campaign roll-ups. Inspect the input shape before constructing it:

```bash
buffer schema describe aggregatedPostMetrics --output json \
  | jq '.jsonInputSchema.properties.tags'
```

### Handling ingestion lag

A defensive aggregate report should:

1. Pull the aggregate.
2. Read `metricsUpdatedAt`.
3. If null → no posts in window, report as such.
4. If older than `now - 36h` → flag the staleness in the report.
5. Otherwise → report numbers as current as of `metricsUpdatedAt`.

```bash
updated=$(echo "$result" | jq -r '.metricsUpdatedAt // empty')
[ -z "$updated" ] && { echo "No posts in window"; exit 0; }
# Node ships with the CLI runtime, so use it for portable ISO-8601 parsing.
lag=$(node -e "process.stdout.write(String(Math.floor((Date.now() - Date.parse(process.argv[1])) / 1000)))" "$updated")
[ "$lag" -gt 129600 ] && echo "warn: metrics last refreshed ${lag}s ago"
```
