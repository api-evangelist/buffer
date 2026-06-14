# Buffer GraphQL API

Buffer provides a GraphQL API for scheduling and publishing social media posts, managing social media channels, handling content ideas, and accessing post engagement metrics across 11 major social media platforms including Instagram, LinkedIn, X (Twitter), TikTok, Facebook, Threads, Pinterest, Bluesky, YouTube, Mastodon, and Google Business Profiles.

**Endpoint:** https://api.buffer.com/graphql

**Documentation:** https://developers.buffer.com/

## References

- Documentation: https://developers.buffer.com/
- GettingStarted: https://developers.buffer.com/
- Authentication: https://support.buffer.com/article/859-does-buffer-have-an-api
- GitHub Organization: https://github.com/bufferapp

## Overview

The Buffer GraphQL API is a live, introspectable GraphQL endpoint. The schema includes 190 custom types across objects, inputs, enums, interfaces, unions, and scalars. Core operations include:

- **Query:** account, channel, channels, posts, post, aggregatedPostMetrics, dailyPostingLimits
- **Mutation:** createPost, editPost, deletePost, createIdea

Key domain types include `Post`, `Channel`, `Organization`, `Account`, `Tag`, `Idea`, `Schedule`, and per-platform metadata types (e.g. `InstagramPostMetadata`, `TwitterPostMetadata`, `YoutubePostMetadata`).
