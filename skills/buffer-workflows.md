# Workflows

Canonical sequences. Read the persisted organization and timezone from config — `buffer init` writes them on first run, so most workflows skip the account fetch entirely.

## Get timezone for scheduling

```bash
tz=$(buffer config get timezone --output json | jq -r '.value')

# Fallback for configs from before timezone was persisted, or when the user
# wants to operate against a different account: fetch and cache it.
if [ -z "$tz" ] || [ "$tz" = "null" ]; then
  tz=$(buffer account --fields timezone --output json | jq -r '.timezone')
fi
```

To list every organization the account can see (e.g. to switch the active one):

```bash
buffer account --fields 'organizations.{id,name}' --output json
```

If the account has more than one organization, list them by name and confirm with the user before continuing.

## List channels for an org

```bash
buffer channels list --fields 'items.{id,service,name}' --output json
```

See [`getting-started.md#trim-noise`](./getting-started.md#trim-noise) for the full `--fields` rules.

## Inspect a channel before scheduling

```bash
buffer channels get --id "$channelId" --output json
```

Required before posting to Pinterest, before `mode: addToQueue` on a channel with no schedule, or whenever per-service `metadata.*` is needed. See [`pitfalls.md`](./pitfalls.md#per-service-minimum-payloads) for the rule set.

## Create a scheduled post (safe pattern)

```bash
# 1. Build payload, validate locally first
payload=$(jq -nc \
  --arg ch "$channelId" \
  --arg t "Hello" \
  '{channelId:$ch, schedulingType:"automatic", mode:"addToQueue", text:$t}')

buffer posts create --json "$payload" --dry-run        # exit 0 = input parses
buffer posts create --json "$payload" --output json    # send for real
```

## Create a post at a specific local time

```bash
# user said "tomorrow 5 PM" → build ISO with the account timezone offset
dueAt="2026-05-06T17:00:00-05:00"   # offset from buffer account.timezone
buffer posts create --channel-id "$channelId" \
  --scheduling-type automatic --mode customScheduled \
  --due-at "$dueAt" --text "Hello"
```

## Create a draft (zero risk)

```bash
buffer posts create --channel-id "$channelId" \
  --scheduling-type automatic --mode addToQueue \
  --text "scratch" --save-to-draft
```

Drafts skip posting-limit checks and never publish. Use freely while iterating.

## Paginate

Relay connections are flattened to `{items, pageInfo: {hasNextPage, endCursor}}`. Loop on the cursor:

```bash
cursor=""
while :; do
  page=$(buffer posts list ${cursor:+--after "$cursor"} --output json)
  echo "$page" | jq -c '.items[]'
  has=$(echo "$page" | jq -r '.pageInfo.hasNextPage')
  cursor=$(echo "$page" | jq -r '.pageInfo.endCursor')
  [ "$has" = "true" ] || break
done
```

See [`pitfalls.md#reading-responses`](./pitfalls.md#reading-responses) for the cursor-opacity rule.

## List idea groups (folders)

```bash
buffer ideaGroups list --organization-id "$orgId" --output json
```

Idea groups are the folders ideas live in. Each returns `id`, `name`, and
`isLocked`. List them before listing or creating ideas when you need a group
`id` — there is no separate get-by-id command. This is a plain list, not a
Relay connection, so there are no pagination flags.

## Capture an idea, then promote it later

```bash
ideaResult=$(buffer ideas create --text "Tweet about launch" --output json)
ideaId=$(echo "$ideaResult" | jq -r '.idea.id')

# later
buffer posts create --channel-id "$channelId" \
  --scheduling-type automatic --mode addToQueue \
  --idea-id "$ideaId"
```

## Check posting limits before bulk scheduling

```bash
buffer dailyPostingLimits list \
  --channel-ids "$id1,$id2,$id3" \
  --date "2026-05-05" --output json
```

Single-org constraint applies — see [`pitfalls.md`](./pitfalls.md#cross-org-operations).

## Delete a post

```bash
buffer posts delete --id "$postId"      # second call → exit 3 (already gone)
```
