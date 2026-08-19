# Buffer GraphQL API

Buffer provides a GraphQL API for scheduling and publishing social media posts, managing social media channels, handling content ideas and idea groups, post templates, and accessing post engagement metrics across 11 major social media platforms including Instagram, LinkedIn, X (Twitter), TikTok, Facebook, Threads, Pinterest, Bluesky, YouTube, Mastodon, and Google Business Profiles.

**Endpoint:** https://api.buffer.com (the same GraphQL service also answers at https://api.buffer.com/graphql)

**Authentication:** `Authorization: Bearer <token>` — a personal API key or an OAuth 2.0 access token. Anonymous requests return HTTP 401 with `extensions.code: UNAUTHENTICATED`.

**Documentation:** https://developers.buffer.com/

## Artifacts in this directory

| File | What it is | Provenance |
|---|---|---|
| `buffer-schema.graphql` | Introspected SDL snapshot, 190 custom types | captured 2026-06; **now behind the live schema** |
| `buffer-api-reference.md` | Buffer's own complete published API reference | fetched verbatim 2026-08-13 from https://developers.buffer.com/reference.md |

> **The SDL snapshot is stale and the reference is not.** Live introspection is
> auth-gated (`POST {"query":"{__typename}"}` → 401), so the SDL could not be
> refreshed anonymously. Buffer's published reference was fetched instead and is
> the authoritative current shape. Operations present in the reference but
> **missing** from the SDL snapshot: `Query.ideas`, `Query.ideaGroups`,
> `Query.postTemplate`, `Query.postTemplates`, `Mutation.createPostTemplate`,
> `Mutation.updatePostTemplate`, `Mutation.deletePostTemplate`,
> `Mutation.movePostInQueue`. See `changelog/buffer-changelog.yml` for when each
> landed.

## References

- Developer portal: https://developers.buffer.com/
- API reference: https://developers.buffer.com/reference.html
- Getting started: https://developers.buffer.com/guides/getting-started.html
- Authentication + OAuth: https://developers.buffer.com/guides/authentication.html
- API standards: https://developers.buffer.com/guides/api-standards.html
- Changelog: https://developers.buffer.com/changelog.html
- Roadmap: https://developers.buffer.com/roadmap.html
- API Explorer: https://developers.buffer.com/explorer.html
- llms.txt: https://developers.buffer.com/llms.txt
- GitHub organization: https://github.com/bufferapp

## Operations (from the published reference, 2026-08-13)

**Queries (10):** `account`, `aggregatedPostMetrics`, `channel`, `channels`, `dailyPostingLimits`, `ideaGroups`, `ideas`, `post`, `posts`, `postTemplate`, `postTemplates`

**Mutations (8):** `createIdea`, `createPost`, `createPostTemplate`, `deletePost`, `deletePostTemplate`, `editPost`, `movePostInQueue`, `updatePostTemplate`

**No subscriptions.** The schema declares only `schema { query, mutation }`. Buffer publishes no webhooks and no streaming surface — see `conformance/buffer-conformance.yml`.

Maturity markers Buffer publishes on the reference: post templates are **Preview**, `movePostInQueue` is **Experimental**, and the metrics schema is annotated `@experimental`.

## Domain types

Core entities are `Account`, `Organization`, `Channel`, `Post`, `Idea`, `IdeaGroup`, `PostTemplate`, `Tag`, `Note` and `Asset`, plus per-platform metadata unions (`PostMetadata` over eleven types, `ChannelMetadata` over ten). Every entity id is a dedicated MongoDB-ObjectId scalar (`PostId`, `ChannelId`, `OrganizationId`, …). The full entity-relationship graph is derived in `data-model/buffer-data-model.yml`.

## Conventions

Cursor pagination follows the Relay convention (`first`/`after`, `edges`, `pageInfo`), forward-only. Errors use a two-category model: typed union members implementing `MutationError` for recoverable problems, and a standard `errors[]` array with `extensions.code` for non-recoverable ones — HTTP is 200 either way. There is **no idempotency-key mechanism**. See `conventions/buffer-conventions.yml` and `errors/buffer-error-codes.yml`.
