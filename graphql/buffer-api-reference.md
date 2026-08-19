# Buffer API Reference

> Connect Buffer to your agents, automation tools, or build something entirely new.

API Endpoint: https://api.buffer.com
Authentication: Bearer token via Authorization header

## Queries

#### account

Retrieves the authenticated user's account information

**Returns:** `Account!`

#### aggregatedPostMetrics

Aggregate normalized post metrics across a filtered post set. Useful
for yearly summaries, channel-level rollups, and BI exports without
paginating through thousands of posts.

For per-post metrics, use `posts(input)` or `post(input)` with a
`metrics { … }` selection — this query is purely for aggregation.

The result always contains a baseline trio of entries: `postCount`
(number of matched posts in the window), `reactions`, and `comments`.
Posts on networks that don't track reactions or comments contribute 0
to those totals.

Beyond the baseline, additional metric types are returned only when
every channel in the filter set supports them. A single-network filter
surfaces that network's richer metrics (e.g. impressions, reach,
engagementRate on LinkedIn); a mixed-network filter trims the extras
to those common to every network in the set.

**Returns:** `AggregatedPostMetrics!`

**Arguments:**
- `input`: `AggregatedPostMetricsInput!` - Query's input: organization, date range, optional channel and tag
filters. Date range is capped to 365 days.

#### channel

Fetches a single channel using the provided ID

**Returns:** `Channel!`

**Arguments:**
- `input`: `ChannelInput!` - Query's input.

#### channels

Fetch all channels for the organization taking into account the current's user permissions

**Returns:** `[Channel!]!`

**Arguments:**
- `input`: `ChannelsInput!` - Query's input.

#### dailyPostingLimits

Returns daily posting limit status for the given channels on the specified date.

**Returns:** `[DailyPostingLimitStatus!]!`

**Arguments:**
- `input`: `DailyPostingLimitsInput!` - Query's input.

#### ideaGroups

Retrieves idea groups based on the provided input parameters.

**Returns:** `[IdeaGroup!]!`

**Arguments:**
- `input`: `IdeaGroupsInput!` - Input for retrieving idea groups.

#### ideas

Fetch a paginated list of ideas with optional filtering

**Returns:** `IdeasConnection!`

**Arguments:**
- `after`: `String` - Cursor for pagination, marks where to start fetching from
- `first`: `Int` - Maximum number of items to return
- `input`: `IdeasInput!` - Filtering criteria for the ideas list

#### post

Fetches a post by PostID for the given organization: first and last can be set for forward pagination using Relay convention

**Returns:** `Post!`

**Arguments:**
- `input`: `PostInput!` - Query's input.

#### posts

Fetches posts for the given organization: first and last can be set for forward pagination using Relay convention

**Returns:** `PostsResults!`

**Arguments:**
- `after`: `String` - The cursor of the post to start fetching from
- `first`: `Int` - The number of posts to return
- `input`: `PostsInput!` - Query's input.

#### postTemplate

Fetch a single post template by ID. Returns null if not found.

**Status:** 🧪 Preview

**Returns:** `PostTemplate`

**Arguments:**
- `input`: `PostTemplateInput!` - Input for fetching a single post template.

#### postTemplates

Fetch the templates visible to the current actor for the template
library: public templates, plus internal templates from the supplied
`organizationId`, plus private templates owned by the actor's
account. The visibility scope is always pinned to the actor and the
supplied organization — the input filter can only narrow within that
scope, never widen it.

**Status:** 🧪 Preview

**Returns:** `PostTemplatesConnection!`

**Arguments:**
- `after`: `String` - The cursor after which to return results.
- `first`: `Int` - The number of templates to return.
- `input`: `PostTemplatesInput!` - Input containing the organization scope and optional filters.

## Mutations

#### createIdea

Create a new idea with the given content and metadata

**Returns:** `CreateIdeaPayload!`

**Arguments:**
- `input`: `CreateIdeaInput!` - Input to create an idea

#### createPost

Create post for channel

**Returns:** `PostActionPayload!`

**Arguments:**
- `input`: `CreatePostInput!` - The mutation's input

#### createPostTemplate

Create a post template visible only to the caller (`private`) or to
the caller's organization (`internal`).

**Status:** 🧪 Preview

**Returns:** `CreatePostTemplatePayload!`

**Arguments:**
- `input`: `CreatePostTemplateInput!` - Input for creating a post template.

#### deletePost

Delete a post by id.

**Returns:** `DeletePostPayload!`

**Arguments:**
- `input`: `DeletePostInput!` - Input for the deletePost mutation.

#### deletePostTemplate

Delete a post template owned by the caller (or an internal template
in the caller's organization, if the caller is an org admin/owner).

**Status:** 🧪 Preview

**Returns:** `DeletePostTemplatePayload!`

**Arguments:**
- `input`: `DeletePostTemplateInput!` - Input for deleting a post template.

#### editPost

Edit post for channel

**Returns:** `PostActionPayload!`

**Arguments:**
- `input`: `EditPostInput!` - The mutation's input

#### movePostInQueue

Move a queued post to the top or bottom of its channel's queue. Unlike editPost, this
is a scheduling-only operation that never re-validates the post's content.

**Status:** ⚠️ Experimental

**Returns:** `MovePostInQueuePayload!`

**Arguments:**
- `input`: `MovePostInQueueInput!` - The mutation's input

#### updatePostTemplate

Update a post template owned by the caller (or an internal template
in the caller's organization, if the caller is an org admin/owner).

**Status:** 🧪 Preview

**Returns:** `UpdatePostTemplatePayload!`

**Arguments:**
- `input`: `UpdatePostTemplateInput!` - Input for updating a post template.

## Object Types

#### Account

Account is a representation of a Buffer user.

**Fields:**
- `id`: `ID!` - Unique identifier for the account
- `email`: `String!` - Primary email address for the account
- `backupEmail`: `String` - Backup email address for account recovery
- `avatar`: `String!` - URL to the account's avatar image
- `createdAt`: `DateTime` - Date the account was created in the Core DB. For older customers, it's possible a Publish account existed in the Publish DB for this customer before this date
- `organizations`: `[Organization!]!`
  - Arg `filter`: `OrganizationFilterInput`
- `timezone`: `String` - The account-level timezone - this is used as a default input for streaks, posting plans, and new channel channel connections.
- `name`: `String` - The account name, different from the organization name
- `preferences`: `Preferences` - The accounts preferences
- `connectedApps`: `[ConnectedApp!]` - The connected apps for the account

#### AggregatedPostMetrics

Aggregated post metrics across a filtered post set. Each entry in
`metrics` mirrors the shape of a `Post.metrics` entry; the total number
of matched posts is carried as a regular `PostMetric` entry with
`type: postCount`.

**Fields:**
- `metrics`: `[PostMetric!]!` - Normalized metric aggregates across the matched posts. Always includes
a baseline trio (`postCount`, `reactions`, `comments`) — posts on
networks that don't track reactions or comments contribute 0 to those
totals. Beyond the baseline, additional metric types are included only
when every channel in the filter set supports them.
- `metricsUpdatedAt`: `DateTime` - The latest `metricsUpdatedAt` across the matched posts, indicating the
freshness of the aggregate. Metrics are refreshed daily, so values can
be up to ~24h behind the source network. Null when no posts matched
the filter.

#### Annotation

Annotation representing all the entities in the text

**Fields:**
- `content`: `String!` - The content of the annotation. Annotations can sometimes be different from the actual text content.
E.g., Mastodon mentions have 'text: @buffer', but includes the server name in the content, 'content: @buffer@threads.net'
- `indices`: `[Int!]!` - The indices of the annotation in the text
- `text`: `String!` - The text representation of the annotation, eg '@buffer'
- `type`: `AnnotationType!` - The type of the annotation
- `url`: `String!` - The URL the annotation points to

#### Author

Represent the author of a post or note.

**Fields:**
- `id`: `AccountId!` - The unique identifier of the author.
- `avatar`: `String!` - The avatar URL of the author.
- `email`: `String!` - The email address of the author.
- `isDeleted`: `Boolean!` - Indicates whether the author is a deleted.
- `name`: `String` - The name of the author. Null if the user has not yet set a name.

#### BlueskyMetadata

Bluesky metadata

**Fields:**
- `serverUrl`: `String!` - The instance of bluesky of the channel

#### BlueskyPostMetadata

Bluesky post metadata

**Implements:** CommonPostMetadata, ThreadedPostMetadata

**Fields:**
- `annotations`: `[Annotation!]!` - Annotations representing entities in the text
- `linkAttachment`: `LinkAttachment` - Link attachment
- `thread`: `[ThreadedPost!]!` - The list of threaded posts (not paginated)
- `threadCount`: `Int!` - The number of threaded posts
- `type`: `PostType!` - The channel-specific type of the post, eg, post, story, reel for Instagram

#### Channel

Channel entity

**Fields:**
- `id`: `ChannelId!` - The ID of the channel
- `allowedActions`: `[ChannelAction!]!` - The allowed actions for the current user
- `avatar`: `String!` - The avatar URL of the channel
- `descriptor`: `String!` - Formatted name of the channel service and type: e.g. 'Twitter Profile' or 'Facebook Page'
- `displayName`: `String` - The display name of the channel - nullable (reason?)
- `externalLink`: `String` - The channel's URL on the social network (e.g. instagram.com/username or facebook.com/page)
Returns null if the channel is not supported
- `hasActiveMemberDevice`: `Boolean!` - Whether at least one member of the orginization who have access to this channel
also has a user device registered for push notifications
- `isDisconnected`: `Boolean!` - Indicates if the channel is properly connected to Buffer
- `isLocked`: `Boolean!` - Indicates if the channel is locked - Locked channels can't be used for posting.
A channel can be locked when the organization downgrades and reduces the channel quantity of their plan.
- `isNew`: `Boolean!` - Indicates if the channel was recently created (in less than 10 seconds). This is used to determine the redirect modal after channel authorization
- `isQueuePaused`: `Boolean!` - Indicates is the queue is paused for the channel. A paused queue means schedules posts won't be published.
- `linkShortening`: `ChannelLinkShortening!` - Link Shortening settings for the channel
- `metadata`: `ChannelMetadata` - Metadata or settings depending on the service type - such as the server URL for Mastodon or Location data for Facebook/GPB
- `name`: `String!` - The name of the channel - the handle name, username, etc.
- `organizationId`: `OrganizationId!` - The organization ID of the channel
- `postingGoal`: `PostingGoal` - The posting goal for the channel
- `postingSchedule`: `[ScheduleV2!]!` - Provides the posting slots for each day of the week
- `products`: `[Product!]` - Products that support a given channel
- `scopes`: `[String]!` - Scopes requested for a given channel - empty array if we don't have them tracked
- `service`: `Service!` - Represents the social network
- `serviceId`: `String!` - Represents the external ID of the channel on social network API
- `showTrendingTopicSuggestions`: `Boolean!` - Indicates if trending topic suggestions should be shown in the composer.
When false, users can still access trends via the trending icon button.
Defaults to true for backward compatibility.
- `timezone`: `String!` - The timezone of the channel - Default if not set is Europe/London
- `type`: `ChannelType!` - The type of the channel - Page, Profile, Business, Group, Account, etc.
- `weeklyPostingLimit`: `WeeklyPostingLimit` - Weekly posting limit for the channel *(Deprecated: This field is not used anymore)*
- `createdAt`: `DateTime!` - The creation date of the channel
- `updatedAt`: `DateTime!` - The last time the channel was updated

#### ChannelLinkShortening

Settings for link shortening

**Fields:**
- `config`: `LinkShorteningConfig` - Configuration of link shortening integration. Null if disabled.
- `isEnabled`: `Boolean!` - If link shortening is enabled for the channel

#### ConnectedApp

Connected App

**Fields:**
- `category`: `ConnectedAppCategory` - The category of the connected app, when known.
- `clientId`: `ID!` - The id of the connectedApp.
- `description`: `String!` - A brief description of the connected app.
- `name`: `String!` - The name of the connected app.
- `scopes`: `[String!]!` - The access scopes granted to this connection for Buffer's public API
resources. Empty when the connection holds none.
- `userId`: `ID!` - The id of the user that has granted access to the app.
- `website`: `String!` - The website URL of the connected app.
- `createdAt`: `DateTime!` - The date and time when the connected app was created.

#### CreatePostTemplateSuccess

Successful result of an end user creating a post template.

**Status:** 🧪 Preview

**Fields:**
- `postTemplate`: `PostTemplate!` - The newly created post template.

#### DailyPostingLimitStatus

Status of daily posting limits for a channel on a given day.

**Fields:**
- `channelId`: `ChannelId!` - The channel ID this status refers to.
- `isAtLimit`: `Boolean!` - Whether the channel has reached its daily posting limit.
- `limit`: `Int` - The network daily posting limit. Null means unlimited.
- `scheduled`: `Int!` - Number of posts scheduled for this day.
- `sent`: `Int!` - Number of posts already sent on this day.

#### DeletePostSuccess

deletePost success response returns the post id that was deleted.

**Fields:**
- `id`: `PostId!` - Post id that was delete.

#### DocumentAsset

Document asset

**Implements:** Asset

**Fields:**
- `id`: `ID` - The ID of the asset in the database
- `document`: `DocumentMetadata!` - Document specific metadata
- `mimeType`: `String!` - The MIME type of the asset
- `source`: `String!` - URL to the file source
- `thumbnail`: `String!` - URL to the static thumbnail of the asset
- `type`: `AssetType!` - The type of the asset

#### DocumentMetadata

Document metadata

**Fields:**
- `filesize`: `Int` - Document fileSize in bytes
- `numPages`: `Int!` - Number of pages in the document
- `thumbnails`: `[String!]!` - URLs to the static thumbnails of the document pages
- `title`: `String` - Document title

#### EmptySuccess

Empty mutation success response used when client doesn't need any data back
or simply needs to respond to a success or error.

**Implements:** MutationSuccess

**Fields:**
- `_empty`: `String!` - The value is always an empty string ''
Note: GraphQL doesn't allow types with no fields, so we have to add this field

#### FacebookMetadata

Facebook metadata

**Fields:**
- `locationData`: `LocationData` - Metadata about the location of the business associated with the channel. Only available for Facebook and GPB

#### FacebookPostMetadata

Facebook post metadata

**Implements:** CommonPostMetadata

**Fields:**
- `annotations`: `[Annotation!]!` - Annotations representing entities in the text
- `firstComment`: `String` - Facebook post's first comment
- `linkAttachment`: `LinkAttachment` - Link attachment
- `title`: `String` - Title of Facebook reel
- `type`: `PostType!` - The channel-specific type of the post, eg, post, story, reel for Instagram

#### GoogleBusinessEventMetaData

Metadata for a GBP post that is an event

**Fields:**
- `button`: `GoogleBusinessPostActionType!` - Action button
- `endDate`: `DateTime!` - End date of the event
- `endTime`: `String` - End time of the event *(Deprecated: get time from the endDate)*
- `isFullDayEvent`: `Boolean!` - Indicate whether the event has a start or end time.
- `link`: `String` - Link to the action
- `startDate`: `DateTime!` - Start date of the event
- `startTime`: `String` - Start time of the event *(Deprecated: get time from the startDate)*
- `title`: `String!` - Title of the event

#### GoogleBusinessMetadata

Google Business metadata

**Fields:**
- `locationData`: `LocationData` - Metadata about the location of the business associated with the channel. Only available for Facebook and GPB

#### GoogleBusinessOfferMetaData

Metadata for a GBP post that is an offer

**Fields:**
- `code`: `String` - Coupon code for the offer
- `endDate`: `DateTime!` - End date of the offer
- `link`: `String` - Link to the offer
- `startDate`: `DateTime!` - Start date of the offer
- `terms`: `String` - Terms and Conditions
- `title`: `String!` - Title of the offer

#### GoogleBusinessPostMetadata

Google Business Profile post metadata
@deprecated: pending proposal for specific GBP post types: update, offer and event metadata types

**Implements:** CommonPostMetadata

**Fields:**
- `annotations`: `[Annotation!]!` - Annotations representing entities in the text
- `details`: `GoogleBusinessPostDetails` - Details of the metadata
- `title`: `String` - Title if available in the given GBP post type: event and offer
- `type`: `PostType!` - The channel-specific type of the post, eg, post, story, reel for Instagram

#### GoogleBusinessWhatsNewMetaData

Metadata for a GBP post of type Whats new

**Fields:**
- `button`: `GoogleBusinessPostActionType!` - Action button
- `link`: `String` - Link to the action

#### Idea

Ideas are the main entity in the create space

**Fields:**
- `id`: `ID!` - Unique identifier for the idea
- `content`: `IdeaContent!` - The actual content and metadata of the idea
- `groupId`: `ID` - ID of the group this idea belongs to (if any)
- `organizationId`: `ID!` - ID of the organization that owns this idea
- `position`: `Float` - Numerical position for ordering within a group
- `createdAt`: `Int!` - Unix timestamp of when the idea was created
- `updatedAt`: `Int!` - Unix timestamp of when the idea was last modified

#### IdeaContent

Content of an idea

**Fields:**
- `aiAssisted`: `Boolean!` - Indicates whether AI tools were used in creating this idea
- `date`: `DateTime` - DateTime set by user associated with the idea - this often reflects a target publish date.
- `media`: `[IdeaMedia!]` - List of media items attached to the idea
- `services`: `[Service!]!` - Services tagged by the user - this is typically used to annotate ideas with their target services
- `tags`: `[PublishingTag!]!` - Tags used to categorize and organize the idea
- `text`: `String` - Main body text or description of the idea
- `title`: `String` - Title or headline of the idea

#### IdeaEdge

Pagination type for Ideas

**Fields:**
- `cursor`: `String!` - Opaque cursor for pagination, used to fetch subsequent pages
- `node`: `Idea!` - The idea object

#### IdeaGroup

Idea groups are used to organize ideas in the board

**Fields:**
- `id`: `ID!` - Unique identifier for the idea group.
- `isLocked`: `Boolean!` - Whether the idea group is locked.
- `name`: `String!` - The name of the idea group.

#### IdeaMedia

Media attached to an idea

**Fields:**
- `id`: `ID!` - Unique identifier for the media in Buffer's upload system
- `alt`: `String` - Alternative text description for accessibility
- `size`: `Int` - File size in bytes
- `source`: `IdeaMediaSource` - Source platform information for the media
- `thumbnailUrl`: `String` - URL to a smaller version of the media for preview purposes
- `type`: `MediaType!` - Type of media (e.g., image, video, gif)
- `url`: `String!` - Direct URL to access the media file

#### IdeaMediaSource

Media source for the idea, e.g. Unsplash, Gifphy, etc.

**Fields:**
- `id`: `String` - Unique identifier from the source platform
- `author`: `String` - Name of the content creator/author
- `authorUrl`: `String` - URL to the author's profile on the source platform
- `name`: `String!` - Name of the media source platform (e.g., 'Unsplash', 'Giphy')

#### IdeaResponse

createIdea response type

**Fields:**
- `idea`: `Idea` - The affected idea
- `refreshIdeas`: `Boolean!` - If true, the client should refresh the ideas list because other ideas might have been moved

#### IdeasConnection

Relay connection for paginated ideas.

**Fields:**
- `edges`: `[IdeaEdge!]!` - List of idea edges containing the ideas and their cursors
- `pageInfo`: `PaginationPageInfo!` - Pagination metadata including hasNextPage and endCursor

#### ImageAsset

Image asset

**Implements:** Asset

**Fields:**
- `id`: `ID` - The ID of the asset in the database
- `image`: `ImageMetadata!` - Image specific metadata
- `mimeType`: `String!` - The MIME type of the asset
- `source`: `String!` - URL to the file source
- `thumbnail`: `String!` - URL to the static thumbnail of the asset
- `type`: `AssetType!` - The type of the asset

#### ImageMetadata

Image metadata

**Fields:**
- `altText`: `String!` - Alternative text for accessibility
- `animatedThumbnail`: `String` - Animated thumbnail URL
- `height`: `Int!` - Image height in pixels
- `isAnimated`: `Boolean!` - Is the image animated?
- `userTags`: `[UserTag!]` - User tags in the image
- `width`: `Int!` - Image width in pixels

#### InstagramGeolocation

Instagram Geolocation

**Fields:**
- `id`: `String` - The id of this location
- `text`: `String` - The name of this location

#### InstagramMetadata

Instagram metadata

**Fields:**
- `defaultToReminders`: `Boolean!` - Indicates if we should default to reminder for Instagram

#### InstagramPostMetadata

Instagram post metadata

**Implements:** CommonPostMetadata

**Fields:**
- `annotations`: `[Annotation!]!` - Annotations representing entities in the text
- `firstComment`: `String` - Instagram post's first comment
- `geolocation`: `InstagramGeolocation` - Geolocation of the post
- `isAiGenerated`: `Boolean!` - Whether the post discloses AI-generated content
- `link`: `String` - Shop Grid link for the post
- `shouldShareToFeed`: `Boolean!` - Indicates whether post should be shared to feed
- `stickerFields`: `InstagramStickerFields` - Sticker fields for reminder-based publishing
- `type`: `PostType!` - The channel-specific type of the post, eg, post, story, reel for Instagram

#### InstagramStickerFields

Instagram fields for reminder-based publishing. Upon the reminder for publishing, the user
is prompted to copy and paste these fields into the Instagram app to complete the post.

**Fields:**
- `music`: `String` - Placeholder text for the post's music
- `other`: `String` - Additional field for any other post content
- `products`: `String` - Placeholder text for the post's linked products
- `text`: `String` - Text for the Story or Reel
- `topics`: `String` - Placeholder text for the post's topics (Reels only)

#### InvalidInputError

Error returned when the input is invalid

**Implements:** MutationError

**Fields:**
- `message`: `String!` - Error message

#### LimitReachedError

Error returned when the limit is reached

**Implements:** MutationError

**Fields:**
- `message`: `String!` - Error message

#### LinkAttachment

Link attachment

**Implements:** ScrapedLink

**Fields:**
- `expandedUrl`: `String` - Full URL that the link asset has been built from
- `text`: `String!` - Description for the scraped link
- `thumbnail`: `String` - Selected thumbnail for this link preview
- `thumbnails`: `[String!]!` - Thumbnails of media available in the link
- `title`: `String!` - Title for the link attachment
- `url`: `String!` - URL that the link asset has been built from

#### LinkedInMetadata

LinkedIn metadata

**Fields:**
- `shouldShowLinkedinAnalyticsRefreshBanner`: `Boolean!` - Property parsed from scopes indicating whether the client should show the LinkedIn analytics refresh banner

#### LinkedInPostMetadata

LinkedIn post metadata

**Implements:** CommonPostMetadata

**Fields:**
- `annotations`: `[Annotation!]!` - Annotations representing entities in the text
- `firstComment`: `String` - LinkedIn post's first comment
- `linkAttachment`: `LinkAttachment` - Link attachment
- `type`: `PostType!` - The channel-specific type of the post, eg, post, story, reel for Instagram

#### LinkShorteningConfig

Link Shortening Configuration

**Fields:**
- `domain`: `String!` - Domain of the link shortener - eg buff.ly, dub.co,
or the user's custom domain.
- `name`: `String!` - Human readable string to describe the link shortening
service.

#### LocationData

Location data about the channel

**Fields:**
- `googleAccountId`: `String` - Google Account Id of the business
- `location`: `String` - Location of the business associated with the channel
- `mapsLink`: `String` - Link to the map

#### MastodonMetadata

Mastodon metadata

**Fields:**
- `maxCharacters`: `Int!` - Maximum character limit allowed for this channel's server instance
- `serverUrl`: `String!` - The instance of mastodon of the channel

#### MastodonPostMetadata

Mastodon post metadata

**Implements:** CommonPostMetadata, ThreadedPostMetadata

**Fields:**
- `annotations`: `[Annotation!]!` - Annotations representing entities in the text
- `spoilerText`: `String` - Spoiler text hiding the root text of this post
- `thread`: `[ThreadedPost!]!` - The list of threaded posts (not paginated)
- `threadCount`: `Int!` - The number of threaded posts
- `type`: `PostType!` - The channel-specific type of the post, eg, post, story, reel for Instagram

#### MemberConnection

Represents the members connection edge. Later, we can add the list of members with the page info to follow our connection edge pattern.

**Fields:**
- `totalCount`: `Int!` - The total count of team members counting the org owner and team members from the Publish DB.

#### Note

Note entity

**Fields:**
- `id`: `NoteId!` - The unique identifier of the note.
- `allowedActions`: `[NoteAction!]!` - The allowed actions a user can perform on the note.
- `author`: `Author!` - The author of the note - null if the user is deleted or left the organization.
- `text`: `String!` - The content of the note.
- `type`: `NoteType!` - The type of the note.
- `createdAt`: `DateTime!` - The date and time when the note was created.
- `updatedAt`: `DateTime` - The date and time when the note was last edited.

#### NotFoundError

Error returned when the resource is not found

**Implements:** MutationError

**Fields:**
- `message`: `String!` - Error message

#### Organization

Organization is a representation of a Buffer Organization.

**Fields:**
- `id`: `OrganizationId!` - The ID of the organization.
- `channelCount`: `Int!` - The total number of channels connected to the organization.
- `limits`: `OrganizationLimits!` - The limits of the organization. Can be used to check if the organization has reached the limit of channels, members, etc.
- `members`: `MemberConnection!` - The members of the organization. Can be used to check the total number of members in the organization. In the future, it might contain more information about the members.
- `name`: `String!` - The name of the organization.
- `ownerEmail`: `String!` - The owner email of the organization.
- `shouldEnforce2FASetup`: `Boolean!` - Whether the requesting actor should be sent through 2FA setup before they
can use this organization. Derived: true only when the org has
`settings.enforce2FA` ON, the `organization-enforced-2fa` rollout Split is
ON for the org, and the actor has no 2FA configured on their own account.

This is the single source of truth for the forced-2FA gate; app-shell and
publish-frontend redirect to the setup flow when it's true. Exposed to the
API gateway so any stitched consumer reads the same value.

#### OrganizationLimits

Resource limits for an organization including channels, members, and content limits

**Fields:**
- `channels`: `Int!` - The maximum number of channels allowed for the organization.
- `generateContent`: `Int!` - The maximum number of content generations allowed for the organization.
- `ideaGroups`: `Int!` - The maximum number of idea groups allowed for the organization.
- `ideas`: `Int!` - The maximum number of ideas allowed for the organization.
- `members`: `Int!` - The maximum number of members allowed for the organization.
- `postTemplates`: `Int!` - The maximum number of post templates allowed for the organization.
- `savedReplies`: `Int!` - The maximum number of saved replies allowed for the organization.
- `scheduledPosts`: `Int!` - The maximum number of scheduled posts allowed for the organization.
- `scheduledStoriesPerChannel`: `Int!` - The maximum number of scheduled stories allowed per channel.
- `scheduledThreadsPerChannel`: `Int!` - The maximum number of scheduled threads allowed per channel.
- `tags`: `Int!` - The maximum number of tags allowed for the organization.

#### PaginationPageInfo

Information to aid in pagination.

**Fields:**
- `endCursor`: `String` - The last cursor in the list. It can be used to fetch the next page.
- `hasNextPage`: `Boolean!` - When set to true, it means there is a next page available.
- `hasPreviousPage`: `Boolean!` - When set to true, it means there is a previous page available. Will always return false for now as we only support forward pagination.
- `startCursor`: `String` - The first cursor in the list. It can be used to fetch the previous page.

#### PinterestBoard

A Pinterest board

**Fields:**
- `id`: `String!` - The ID of the board
- `avatar`: `String` - The board avatar
- `description`: `String` - The board description
- `name`: `String!` - The board name
- `serviceId`: `String!` - The ID of the service
- `url`: `String!` - The board URL

#### PinterestMetadata

Pinterest metadata

**Fields:**
- `boards`: `[PinterestBoard!]!` - The list of boards the user has on Pinterest

#### PinterestPostMetadata

Pinterest post metadata

**Implements:** CommonPostMetadata

**Fields:**
- `annotations`: `[Annotation!]!` - Annotations representing entities in the text
- `board`: `PinterestBoard` - The board the Pin is saved to
- `title`: `String` - The title of the Pin
- `type`: `PostType!` - The channel-specific type of the post, eg, post, story, reel for Instagram
- `url`: `String` - The Pin destination link

#### Post

Post entity

**Fields:**
- `id`: `PostId!` - ObjectId of the post
- `allowedActions`: `[PostAction!]!` - Indicates what actions the current account can perform on the post
- `assets`: `[Asset!]!` - assets
- `author`: `Author` - Represents the user who created the post
- `channel`: `Channel!` - channel
- `channelId`: `ChannelId!` - channel ID (faster than resolving the channnel.id)
- `channelService`: `Service!` - channel service (faster than resolving the channnel.service)
- `dueAt`: `DateTime` - Date when the post is scheduled to be published
- `error`: `PostPublishingError` - error
- `externalLink`: `String` - The external URL of the post at the destination service
- `ideaId`: `IdeaId` - Is set when the Post is generated from an Idea
- `isCustomScheduled`: `Boolean!` - Indicates whether time to publish was manually selected by the user
- `metadata`: `PostMetadata` - Metadata of the post which differs based on the social network/service
@see post.metadata.graphql
- `metrics`: `[PostMetric!]` - Metrics for the sent post. If post is not yet sent, this field will be null
- `metricsUpdatedAt`: `DateTime` - Timestamp of when `metrics` were last refreshed from the network. Null
until the daily ingestion job has processed the post. Buffer pulls
fresh metrics once per day, so this can lag the network value by ~24h.
- `notes`: `[Note!]!` - notes
- `notificationStatus`: `NotificationStatus` - notificationStatus: notified or markedAsPublished
- `schedulingType`: `SchedulingType` - How the post publishes: `notification` for a reminder that asks someone to
publish it by hand, `automatic` for one the publishing workers send.
- `sentAt`: `DateTime` - Date when the post is published
- `sharedNow`: `Boolean!` - Indicates whether the post was shared via publish now action
- `shareMode`: `ShareMode!` - Indicates the share mode of the post (e.g., addToQueue, shareNext, shareNow, customScheduled)
- `status`: `PostStatus!` - status
- `tags`: `[Tag!]!` - tags - sorted by name in ascending order
- `text`: `String!` - Text content of the Post
- `via`: `PostVia!` - Indicates if the post is created from Buffer or the API
- `createdAt`: `DateTime!` - Date when the post was created
- `updatedAt`: `DateTime!` - Date when the post was updated

#### PostActionSuccess

Success response returns the full up-to-date post from after the action was performed.

**Fields:**
- `post`: `Post!` - Post on which the action was successfully performed.

#### PostingGoal

Represents a posting goal for a channel, including target, progress, and status information.

**Fields:**
- `goal`: `Int!` - The target number of posts for this goal.
- `periodEnd`: `DateTime!` - The end date of the period for this posting goal.
- `periodStart`: `DateTime!` - The start date of the period for this posting goal.
- `scheduledCount`: `Int!` - The number of posts that are scheduled to be sent for this goal.
- `sentCount`: `Int!` - The number of posts that have been sent (published or ingested) for this goal.
- `status`: `PostingGoalStatus!` - The current status of the posting goal.

#### PostMetric

A single metric for a post (or an entry in an aggregated metrics response).

**Fields:**
- `description`: `String!` - A human-readable description of what the metric represents.
- `name`: `String!` - The human-readable name of the metric (e.g. "Reactions", "Eng. Rate").
- `type`: `PostMetricType!` - The type of metric. Cross-network metrics use unified naming (`reactions`,
`comments`, etc.); network-specific metrics retain their network's
vocabulary (`saves`, `quotes`, etc.). See `PostMetricType` for the full
catalog including deprecated values.
- `unit`: `PostMetricUnit!` - The unit (count vs percentage) of `value`.
- `value`: `Float!` - The metric value. Defaults to 0 when the network did not report the metric.

#### PostPublishingError

Post publishing error

**Fields:**
- `message`: `String!` - Error message to display
- `rawError`: `String` - The original error from the publishing service (internal use only)
- `supportUrl`: `String` - Link to a help center article to help resolve the error

#### PostsEdge

Represent a node in the pagination result using the Connect Relay convention.

**Fields:**
- `cursor`: `String!` - Opaque cursor to be used in pagination to fetch from current node.
- `node`: `Post!` - Represents the current post in the list.

#### PostsResults

Results for the posts query.

**Fields:**
- `edges`: `[PostsEdge!]` - The list of posts that match the query.
- `pageInfo`: `PaginationPageInfo!` - Information to aid in pagination.

#### PostTemplate

A post template used for content inspiration.

**Fields:**
- `id`: `PostTemplateId!` - The unique identifier for the template. *(🧪 Preview)*
- `body`: `String!` - The main content body of the template, may contain {{placeholders}}. *(🧪 Preview)*
- `description`: `String!` - A short user-facing description of the template. *(🧪 Preview)*
- `emoji`: `String!` - The emoji associated with the template. *(🧪 Preview)*
- `organizationId`: `OrganizationId!` - The organization that owns this template. *(🧪 Preview)*
- `title`: `String!` - The title of the template. *(🧪 Preview)*
- `visibility`: `PostTemplateVisibility!` - The visibility level of the template. `public` is returned for
Buffer-managed templates. *(🧪 Preview)*
- `createdAt`: `DateTime!` - The date and time the template was created. *(🧪 Preview)*
- `updatedAt`: `DateTime!` - The date and time the template was last updated. *(🧪 Preview)*

#### PostTemplateEdge

An edge in a post template connection.

**Status:** 🧪 Preview

**Fields:**
- `cursor`: `String!` - A cursor for pagination.
- `node`: `PostTemplate!` - The post template node at the end of the edge.

#### PostTemplatesConnection

A paginated connection of post templates.

**Status:** 🧪 Preview

**Fields:**
- `edges`: `[PostTemplateEdge!]!` - The list of post template edges.
- `pageInfo`: `PaginationPageInfo!` - Pagination information for the connection.
- `totalCount`: `Int!` - The total number of templates matching the filters.

#### Preferences

Account preferences

**Fields:**
- `timeFormat`: `String`
- `startOfWeek`: `String`
- `defaultScheduleOption`: `ScheduleOption!`

#### PublishingTag

Tag snapshot associated with content

**Fields:**
- `id`: `ID!`
- `color`: `String!` - Hex color for tag e.g #F523F1
- `colorName`: `TagColorName` - Stable Buffer palette identifier derived from `color`.
Clients should map this value to platform- and theme-specific presentation,
falling back to `color` when this field is null or unsupported.
Returns null for custom or unrecognized colors.
- `name`: `String!`

#### RestProxyError

Error proxied from the REST API response

**Implements:** MutationError

**Fields:**
- `code`: `Int` - Error code from the REST API response - https://buffer.com/developers/api/errors
- `link`: `String` - Link to our Help center from the REST API response
- `message`: `String!` - An error message from the REST API response that we proxied here

#### RetweetMetadata

Information about the initial Tweet that was retweeted

**Implements:** ScrapedLink

**Fields:**
- `id`: `String!` - Retweet ID
- `text`: `String!` - Text of the original tweet
- `thumbnails`: `[String!]!` - Thumbnails to media available in the link
- `url`: `String!` - Link to original tweet
- `user`: `RetweetUserMetadata!` - User who created the original tweet
- `createdAt`: `DateTime!` - Date when the original tweet was created

#### RetweetUserMetadata

Information about the initial author of the Tweet that was retweeted

**Fields:**
- `avatar`: `String!` - Avatar of the user who created the original Tweet
- `name`: `String!` - Name of the user who created the original Tweet
- `username`: `String!` - Username of the user who created the original Tweet

#### ScheduleV2

Posting schedule for a specific day of the week

**Fields:**
- `day`: `DayOfWeek!` - The day of the week: mon, tue, wed, thu, fri, sat, sun
- `paused`: `Boolean!` - Indicates if this day is paused in the posting schedule.
- `times`: `[String!]!` - The times the channel is scheduled to post on the day: HH:MM

#### Tag

Tag entity

**Fields:**
- `id`: `TagId!` - ObjectId of the tag
- `color`: `String!` - Hex color for tag e.g '#F523F1'
- `colorName`: `TagColorName` - Stable Buffer palette identifier derived from `color`.
Clients should map this value to platform- and theme-specific presentation,
falling back to `color` when this field is null or unsupported.
Returns null for custom or unrecognized colors.
- `isLocked`: `Boolean!` - Locked tag cannot be assigned to new items in the UI.
A Tag is locked after a customer downgrades and has more tags than the free plan limit allows
- `name`: `String!` - Name of the tag e.g 'Summer sales'

#### ThreadedPost

A post authored by the user which is posted to a thread.
This is commonly used for long-format twitter and meta threads posts to
allow authored content to span multiple threads.
Threads are represented as a list of replies, each replying to the previous one.

**Fields:**
- `assets`: `[Asset!]!` - Media assets of the threaded post
- `linkAttachment`: `LinkAttachment` - Link attachment for the threaded post
- `text`: `String!` - The text body content of the threaded post

#### ThreadsPostMetadata

Threads post metadata

**Implements:** CommonPostMetadata, ThreadedPostMetadata

**Fields:**
- `annotations`: `[Annotation!]!` - Annotations representing entities in the text
- `linkAttachment`: `LinkAttachment` - Link attachment
- `locationId`: `String` - LocationId associated with the post
- `locationName`: `String` - Location name associated with the post
- `thread`: `[ThreadedPost!]!` - The list of threaded posts (not paginated)
- `threadCount`: `Int!` - The number of threaded posts
- `topic`: `String` - Topic associated with the post
- `type`: `PostType!` - The channel-specific type of the post, eg, post, story, reel for Instagram

#### TiktokMetadata

Tiktok metadata

**Fields:**
- `defaultToReminders`: `Boolean!` - Indicates if we should default to reminder for Tiktok

#### TiktokPostMetadata

Tiktok post metadata

**Implements:** CommonPostMetadata

**Fields:**
- `annotations`: `[Annotation!]!` - Annotations representing entities in the text
- `isAiGenerated`: `Boolean!` - Whether the post discloses AI-generated content (TikTok video only)
- `title`: `String` - The title of the TikTok post (for photo posts)
- `type`: `PostType!` - The channel-specific type of the post, eg, post, story, reel for Instagram

#### TwitterMetadata

Twitter metadata

**Fields:**
- `subscriptionType`: `String` - Indicates the type of subscription the user has on Twitter

#### TwitterPostMetadata

Twitter post metadata

**Implements:** CommonPostMetadata, ThreadedPostMetadata

**Fields:**
- `annotations`: `[Annotation!]!` - Annotations representing entities in the text
- `isAiGenerated`: `Boolean!` - Whether the post discloses AI-generated content
- `retweet`: `RetweetMetadata` - The details of the tweet being retweeted
- `thread`: `[ThreadedPost!]!` - The list of threaded posts (not paginated)
- `threadCount`: `Int!` - The number of threaded posts
- `type`: `PostType!` - The channel-specific type of the post, eg, post, story, reel for Instagram

#### UnauthorizedError

Error returned when the user is not authorized to perform the action

**Implements:** MutationError

**Fields:**
- `message`: `String!` - Error message

#### UnexpectedError

Error returned when unexpected error occurs

**Implements:** MutationError

**Fields:**
- `message`: `String!` - Error message

#### UpdatePostTemplateSuccess

Successful result of an end user updating a post template.

**Status:** 🧪 Preview

**Fields:**
- `postTemplate`: `PostTemplate!` - The updated post template.

#### UserTag

User tag in the image

**Fields:**
- `handle`: `String!` - The handle (username) of the tagged account, without the leading @.
- `x`: `Float!` - Horizontal position of the tag as a normalized decimal float between 0.0 and 1.0 - the fraction of the image width from the left edge (0.5 is the horizontal center).
- `y`: `Float!` - Vertical position of the tag as a normalized decimal float between 0.0 and 1.0 - the fraction of the image height from the top edge (0.5 is the vertical center).

#### VideoAsset

Video asset

**Implements:** Asset

**Fields:**
- `id`: `ID` - The ID of the asset in the database
- `mimeType`: `String!` - The MIME type of the asset
- `source`: `String!` - URL to the file source
- `thumbnail`: `String!` - URL to the static thumbnail of the asset
- `type`: `AssetType!` - The type of the asset
- `video`: `VideoMetadata!` - Video specific metadata

#### VideoMetadata

Video metadata

**Fields:**
- `audioCodec`: `String` - Audio codec
- `containerFormat`: `String` - Video container format
- `durationMs`: `Int!` - Video duration in seconds
- `fileSize`: `Int` - Video fileSize in bytes
- `frameRate`: `Int` - Video framerate
- `height`: `Int!` - Video height in pixels
- `isTranscodingRequired`: `Boolean!` - Whether the video needs to be transcoded before it can be broadcasted
- `isVideoProcessing`: `Boolean!` - Whether the video is currently being processed (transcoding in progress)
- `rotationDegree`: `Int` - Rotation degree
- `thumbnailOffset`: `Int` - Offset of the thumbnail chosen for the video, in ms
- `title`: `String` - Video title
- `videoBitRate`: `Int` - Video bitrate in kbps
- `videoCodec`: `String` - Video codec
- `width`: `Int!` - Video width in pixels

#### VoidMutationError

Error implementation that allows clients to resolve the MutationError on mutations that do not currently have typed errors.
This allows clients to automatically handle errors that may be added to a mutation in future.

Do not directly throw this error, use a custom typed error instead

**Implements:** MutationError

**Fields:**
- `message`: `String!` - Error message

#### WeeklyPostingLimit

Weekly posting limit for a channel

**Fields:**
- `limit`: `Int!` - The weekly posting limit for the channel
- `scheduled`: `Int!` - The number of posts the channel has scheduled for this week
- `sent`: `Int!` - The number of posts the channel has sent this week

#### YoutubeCategory

**Fields:**
- `categoryId`: `String!`
- `title`: `String!`

#### YoutubeMetadata

Youtube metadata

**Fields:**
- `defaultToReminders`: `Boolean!` - Indicates if we should default to reminder for Youtube

#### YoutubePostMetadata

Youtube post metadata

**Implements:** CommonPostMetadata

**Fields:**
- `annotations`: `[Annotation!]!` - Annotations representing entities in the text
- `category`: `YoutubeCategory` - Post category
- `embeddable`: `Boolean!` - Indicates whether the video allows embedding
- `isAiGenerated`: `Boolean!` - Whether the post discloses AI-generated content
- `license`: `YoutubeLicense` - Video license
- `madeForKids`: `Boolean!` - Indicates whether the video is suitable for kids
- `notifySubscribers`: `Boolean!` - Indicates whether to notify subscribers on publish video
- `privacy`: `YoutubePrivacy` - Privacy setting for post
- `title`: `String` - Title of the Youtube post
- `type`: `PostType!` - The channel-specific type of the post, eg, post, story, reel for Youtube

## Input Types

#### AggregatedPostMetricsInput

Input for the `aggregatedPostMetrics` query.

**Fields:**
- `channelIds`: `[ChannelId!]` - Optional list of channel IDs to filter by. When omitted (null), the
aggregate spans every channel in the organization the actor has
insights access to. When set to an empty array, no channels match and
the result is empty.
- `endDateTime`: `DateTime!` - End of the aggregation window. Consumers typically pass UTC midnight of
the last calendar day in the window (the backend treats the range as
inclusive of that day), for example `2026-01-31T00:00:00Z`. Date range
is capped to 365 days.
- `organizationId`: `OrganizationId!` - The organization ID
- `startDateTime`: `DateTime!` - Start of the aggregation window. Consumers typically pass UTC midnight
of the first calendar day in the window, for example
`2026-01-01T00:00:00Z`.
- `tags`: `TagComparator` - Filter to posts with specific tags.
When omitted, all posts are included regardless of tags.

#### AnnotationInputFacebook

Annotation representing all the entities in the text

**Fields:**
- `content`: `String!` - The content of the annotation, e.g. '107509875938399'
- `indices`: `[Int!]!` - The indices of the annotation in the text, e.g. [6, 9] (from 6 to 9 characters in the text)
- `text`: `String!` - The text representation of the annotation, eg 'Buffer'
- `url`: `String!` - The URL the annotation points to, e.g. https://www.facebook.com/107509875938399

#### AnnotationInputLinkedIn

Annotation representing all the entities in the text

**Fields:**
- `id`: `String!` - The id of the annotation, e.g. 1521226
- `entity`: `String!` - The entity of the annotation, e.g. urn:li:organization:1521226
- `length`: `Int!` - The length of the annotation, e.g. 6
- `link`: `String!` - The link of the annotation, e.g. https://www.linkedin.com/company/bufferapp
- `localizedName`: `String!` - The localized name of the annotation, e.g. Buffer
- `start`: `Int!` - The start of the annotation, e.g. 5
- `vanityName`: `String!` - The vanity name of the annotation, e.g. bufferapp

#### AssetInput

A single entity's asset. Exactly one variant must be provided.

**Fields:**
- `document`: `DocumentAssetInput` - Document asset
- `image`: `ImageAssetInput` - Image asset
- `link`: `LinkAssetInput` - Link asset
- `video`: `VideoAssetInput` - Video asset

#### BlueskyPostMetadataInput

Bluesky post metadata

**Fields:**
- `linkAttachment`: `LinkAttachmentInput` - Link attachment. Mutually exclusive with a non-empty `assets` array — input providing both is rejected.
- `thread`: `[ThreadedPostInput!]` - The ordered list of posts that make up the thread (not paginated). This
array is the source of truth for what gets published: every post in the
thread, including the root post, must be provided here. Posts are
published in order, each replying to the previous one. The first item is
the root post and should match the top-level `text` on the post input.

#### ChannelInput

Input for the channel query

**Fields:**
- `id`: `ChannelId!` - The ID of the channel to be retrieved

#### ChannelsFiltersInput

Filter to pass when fetching channels.

**Fields:**
- `isLocked`: `Boolean` - If not defined, it returns all channels
Else,
  if true, it only returns locked channels
  if false, it only returns not locked channels
- `product`: `Product` - If not passed, it return all channels
Else, it filters the channels based on what the product supports.

#### ChannelsInput

Input to pass when fetching channels.

**Fields:**
- `filter`: `ChannelsFiltersInput` - A list of option filters - passing null means we don't want to filter
- `organizationId`: `OrganizationId!` - The Organization id to fetch channels for

#### CreateIdeaInput

createIdea input type

**Fields:**
- `content`: `IdeaContentInput!` - Content and metadata for the new idea
- `cta`: `String` - Call-to-action identifier for analytics tracking
- `group`: `IdeaGroupInput` - Group placement (null for unassigned group)
- `organizationId`: `ID!` - Organization ID that will own the idea
- `templateId`: `String` - Template ID used to create the idea

#### CreatePostInput

Create post's request input.

Note: `metadata.{service}.linkAttachment` is mutually exclusive with a non-empty `assets`
array. Input providing both is rejected.

**Fields:**
- `aiAssisted`: `Boolean` - If this post was created with the help of AI
- `assets`: `[AssetInput!]!` (default: []) - Ordered list of assets on this post.
- `channelId`: `ChannelId!` - Channel's Id for which we want to create the post
- `draftId`: `DraftId` - Is set when the Post is generated from a Draft
- `dueAt`: `DateTime` - Date when the post is scheduled to be published
- `ideaId`: `IdeaId` - Is set when the Post is generated from an Idea
- `metadata`: `PostInputMetaData` - Metadata of the post which differs based on the social network/service
- `mode`: `ShareMode!` - How the post is being scheduled.
- `needsApproval`: `Boolean!` (default: false) - Submit the post for approval instead of scheduling it. A post submitted for
approval is always a draft, so this conflicts with turning `saveToDraft` off.

Only valid when your posting policy on the target channel requires approval.
- `saveToDraft`: `Boolean` - If true, saves the post as a draft instead of scheduling it.
When saving as draft:
- Post status will be 'draft' instead of 'buffer'
- Posting limits are not checked
- The post will not be published until explicitly scheduled
- `schedulingType`: `SchedulingType!` - Scheduling type to indicate notification publishing or automatic publishing
- `source`: `String` - source where the composer was initiated from, used for tracking.
- `tagIds`: `[TagId!]` - List of tag IDs
- `text`: `String` - Text content of the Post.

Note: for threaded posts, this needs to match the first item in the `thread` array.

#### CreatePostTemplateInput

Input for an end user creating a post template. Buffer-curated
taxonomy fields are server-defaulted; setting them is only available
to official Buffer clients.

**Fields:**
- `body`: `String!` - The main content body of the template, may contain {{placeholders}}.
- `description`: `String` - A short user-facing description of the template. Nullable for
backwards-compat at the GraphQL boundary — the resolver rejects
null/empty values with a clear input error so the underlying
storage contract (non-empty string) is still honored.
- `emoji`: `String` - The emoji associated with the template.
- `organizationId`: `OrganizationId!` - Organization the template belongs to. The caller must be a member of
this organization. For `internal` visibility this is the team scope;
for `private` it's recorded on the template but does not affect
visibility.
- `title`: `String!` - The title of the template.
- `visibility`: `PostTemplateVisibility` - Defaults to `private` if omitted. `public` is rejected — it is only
available to official Buffer clients.

#### DailyPostingLimitsInput

Input for the dailyPostingLimits query.

**Fields:**
- `channelIds`: `[ChannelId!]!` - List of channel IDs to check limits for. All channels must belong to the same organization.
- `date`: `DateTime` - The date to check limits for. Defaults to today if not provided.

#### DateTimeComparator

Comparator for filtering by date

**Fields:**
- `end`: `DateTime` - Include results with dates equal to or before
the specified date
- `start`: `DateTime` - Include results with dates equal to or after
the specified date

#### DeletePostInput

deletePost mutation deletes a post by id.

**Fields:**
- `id`: `PostId!` - Post id to delete.

#### DeletePostTemplateInput

Input for an end user deleting a post template.

**Fields:**
- `id`: `PostTemplateId!` - The ID of the template to delete.

#### DocumentAssetInput

Document asset

**Fields:**
- `thumbnailUrl`: `String!` - Document thumbnail URL
- `title`: `String!` - Document title
- `url`: `String!` - Document URL

#### EditPostInput

Edit post's request input.

Note: `metadata.{service}.linkAttachment` is mutually exclusive with a non-empty `assets`
array. Input providing both is rejected.

**Fields:**
- `id`: `PostId!` - ID of the post to edit
- `aiAssisted`: `Boolean` - If this post was edited with the help of AI
- `approvalChange`: `PostApprovalChange` - Change the post's approval state alongside this edit. Leave unset to keep the
post's current approval state.

Only valid when your posting policy on the post's channel requires approval,
and only on your own drafts. Asking for the state the post is already in does
nothing.
- `assets`: `[AssetInput!]` - Ordered list of assets on this post. Omit to preserve the existing list,
pass an empty array to clear it
- `draftId`: `DraftId` - Is set when the Post is generated from a Draft
- `dueAt`: `DateTime` - Date when the post is scheduled to be published
- `ideaId`: `IdeaId` - Is set when the Post is generated from an Idea
- `metadata`: `PostInputMetaData` - Metadata of the post which differs based on the social network/service
- `mode`: `ShareMode` - How the post is being scheduled. Omit the field or pass null to make no
scheduling change — null does not clear or reset the schedule: a scheduled post
keeps its current share mode, queue slot, and any custom time, and the edit
applies only the other provided fields. Pass a non-null ShareMode to apply that
mode.
- `saveToDraft`: `Boolean` - If true, saves the post as a draft instead of keeping it scheduled.
When saving as draft:
- Post status will be 'draft' instead of 'buffer'
- The post will not be published until explicitly scheduled
- `schedulingType`: `SchedulingType` - Scheduling type to indicate notification publishing or automatic publishing.

Omit it, or send null, to leave the post publishing the way it already does.
- `source`: `String` - source where the composer was initiated from, used for tracking.
- `tagIds`: `[TagId!]` - tags
- `text`: `String` - Text content of the Post. Omit the field to keep the current text; pass an
empty string or null to clear it.

Note: for threaded posts, this needs to match the first item in the `thread` array.

#### FacebookPostMetadataInput

Facebook post metadata

**Fields:**
- `annotations`: `[AnnotationInputFacebook!]` - Annotations representing entities in the text
- `firstComment`: `String` - Facebook post's first comment
- `linkAttachment`: `LinkAttachmentInput` - Link attachment. Mutually exclusive with a non-empty `assets` array — input providing both is rejected.
- `type`: `PostTypeFacebook!` - The channel-specific type of the post, eg, post, story, reel for Facebook

#### GoogleBusinessEventMetaDataInput

Metadata for a GBP post that is an event

**Fields:**
- `button`: `GoogleBusinessPostActionType` - Action button. Optional: a post with no button, or `none`, publishes without a call-to-action. On edit, omitting it preserves the existing value.
- `endDate`: `DateTime` - End date of the event. Required on create; optional on edit (omitted preserves existing value).
- `isFullDayEvent`: `Boolean!` - Indicate whether the event has a start or end time.
- `link`: `String` - Link to the action
- `startDate`: `DateTime` - Start date of the event. Required on create; optional on edit (omitted preserves existing value).
- `title`: `String` - Title of the event. Required on create; optional on edit (omitted preserves existing value).

#### GoogleBusinessOfferMetaDataInput

Metadata for a GBP post that is an offer

**Fields:**
- `code`: `String` - Coupon code for the offer
- `endDate`: `DateTime` - End date of the offer. Required on create; optional on edit (omitted preserves existing value).
- `link`: `String` - Link to the offer
- `startDate`: `DateTime` - Start date of the offer. Required on create; optional on edit (omitted preserves existing value).
- `terms`: `String` - Terms and Conditions
- `title`: `String` - Title of the offer. Required on create; optional on edit (omitted preserves existing value).

#### GoogleBusinessPostMetadataInput

Google Business Profile post metadata
@deprecated: pending proposal for specific GBP post types: update, offer and event metadata types

**Fields:**
- `detailsEvent`: `GoogleBusinessEventMetaDataInput` - Details of the Event metadata
- `detailsOffer`: `GoogleBusinessOfferMetaDataInput` - Details of the Offer metadata
- `detailsWhatsNew`: `GoogleBusinessWhatsNewMetaDataInput` - Details of the Whats new metadata
- `title`: `String` - Title if available in the given GBP post type: event and offer
- `type`: `PostTypeGoogleBusiness!` - The channel-specific type of the post, eg, post, offer, event for Google Business Profile

#### GoogleBusinessWhatsNewMetaDataInput

Metadata for a GBP post of type Whats new

**Fields:**
- `button`: `GoogleBusinessPostActionType` - Action button. Optional: a post with no button, or `none`, publishes without a call-to-action. On edit, omitting it preserves the existing value.
- `link`: `String` - Link to the action

#### IdeaContentInput

content input for creating/updating an idea

**Fields:**
- `aiAssisted`: `Boolean` - Whether AI tools were used in creation
- `date`: `DateTime` - Target date for the idea, often used for planning publish schedules
- `media`: `[IdeaMediaInput!]` - List of media items to attach
- `services`: `[Service!]` - Services associated with the idea for targeting specific platforms
- `tags`: `[TagInput!]` - Tags to categorize the idea
- `text`: `String` - Main body text or description
- `title`: `String` - Title or headline of the idea

#### IdeaGroupInput

idea group input for create/update

**Fields:**
- `groupId`: `ID` - Target group ID (null for unassigned group)
- `placeAfterId`: `ID` - ID of idea to place after (null for top position)

#### IdeaGroupsInput

Input type for retrieving idea groups by organization ID.

**Fields:**
- `organizationId`: `ID!` - Unique identifier for the organization.

#### IdeaMediaInput

**Fields:**
- `url`: `String!` - The URL of the media
- `alt`: `String` - Alternative text for the media
- `thumbnailUrl`: `String` - Thumbnail URL for the media
- `type`: `MediaType!` - The type of media (image, gif, video, link, document, unsupported). Note: 'video' is not supported via public API
- `size`: `Int` - The size of the media in bytes
- `source`: `IdeaMediaSourceInput` - Source information for the media

#### IdeaMediaSourceInput

Input type for the source information of media attached to an idea

**Fields:**
- `name`: `String!`
- `id`: `String`
- `trigger`: `String`
- `author`: `String` - for unsplash only
- `authorUrl`: `String`

#### IdeasGroupFilter

Selects which ideas to return by group membership. Exactly one field must be
provided (enforced by @oneOf). To return ideas from all groups, omit
`groupFilter` on IdeasInput rather than setting a field here.

**Fields:**
- `groups`: `[ID!]` - Return only ideas that belong to these specific groups (union/OR).
- `membership`: `IdeaGroupMembership` - Return ideas by a group-membership bucket rather than by specific group IDs.

#### IdeasInput

Filtering criteria for the ideas list.

**Fields:**
- `groupFilter`: `IdeasGroupFilter` - Filter ideas by group membership. Omit this field entirely to return ideas
across all groups.
- `organizationId`: `OrganizationId!` - The organization to fetch ideas from.
- `tagsFilter`: `TagComparator` - Filter ideas by tags using TagComparator.

#### ImageAssetInput

Image asset

**Fields:**
- `metadata`: `ImageMetadataInput` - Image specific metadata
- `thumbnailUrl`: `String` - URL to the static thumbnail of the asset
- `url`: `String!` - URL to the file source

#### ImageDimensionsInput

Image dimensions

**Fields:**
- `height`: `Int!` - Image height in pixels
- `width`: `Int!` - Image width in pixels

#### ImageMetadataInput

Image metadata

**Fields:**
- `altText`: `String!` - Alternative text for accessibility
- `animatedThumbnail`: `String` - Animated thumbnail URL
- `dimensions`: `ImageDimensionsInput` - Ignored — the API resolves image dimensions itself.
- `userTags`: `[UserTagInput!]` - Accounts to tag at specific points on the image. Each tag's x/y position uses normalized 0.0-1.0 coordinates - see UserTagInput.

#### InstagramGeolocationInput

Instagram Geolocation

**Fields:**
- `id`: `String` - The id of this location
- `text`: `String` - The name of this location

#### InstagramPostMetadataInput

Instagram post metadata

**Fields:**
- `firstComment`: `String` - Instagram post's first comment
- `geolocation`: `InstagramGeolocationInput` - Geolocation of the post
- `isAiGenerated`: `Boolean` - Whether the post discloses AI-generated content
- `link`: `String` - Shop Grid link for the post
- `shouldShareToFeed`: `Boolean!` - Indicates whether post should be shared to feed
- `stickerFields`: `InstagramStickerFieldsInput` - Sticker fields for reminder-based publishing
- `type`: `PostType!` - The channel-specific type of the post, eg, post, story, reel for Instagram

#### InstagramStickerFieldsInput

Instagram fields for reminder-based publishing. Upon the reminder for publishing, the user
is prompted to copy and paste these fields into the Instagram app to complete the post.

**Fields:**
- `music`: `String` - Placeholder text for the post's music
- `other`: `String` - Additional field for any other post content
- `products`: `String` - Placeholder text for the post's linked products
- `text`: `String` - Text for the Story or Reel
- `topics`: `String` - Placeholder text for the post's topics (Reels only)

#### LinkAssetInput

Link attached to the post

**Fields:**
- `description`: `String` - Description of the link
- `thumbnailUrl`: `String` - Thumbnail URL of the link
- `title`: `String` - Title of the link
- `url`: `String!` - URL to the link

#### LinkAttachmentInput

Link attachment

**Fields:**
- `description`: `String` - Description shown on the link card
- `thumbnail`: `LinkAttachmentThumbnailInput` - Thumbnail shown on the link card
- `title`: `String` - Title shown on the link card
- `url`: `String!` - URL that the link asset has been built from

#### LinkAttachmentThumbnailInput

Thumbnail of a link attachment

**Fields:**
- `url`: `String!` - URL of the thumbnail image

#### LinkedInPostMetadataInput

LinkedIn post metadata

**Fields:**
- `annotations`: `[AnnotationInputLinkedIn!]` - Annotations representing entities in the text
- `firstComment`: `String` - LinkedIn post's first comment
- `linkAttachment`: `LinkAttachmentInput` - Link attachment. Mutually exclusive with a non-empty `assets` array — input providing both is rejected.

#### MastodonPostMetadataInput

Mastodon post metadata

**Fields:**
- `spoilerText`: `String` - Spoiler text hiding the root text of this post
- `thread`: `[ThreadedPostInput!]` - The ordered list of posts that make up the thread (not paginated). This
array is the source of truth for what gets published: every post in the
thread, including the root post, must be provided here. Posts are
published in order, each replying to the previous one. The first item is
the root post and should match the top-level `text` on the post input.

#### MovePostInQueueInput

movePostInQueue mutation moves a queued post to the top or bottom of its channel's queue.

**Fields:**
- `id`: `PostId!` - ID of the post to move.
- `position`: `QueuePosition!` - Target position within the channel's queue.

#### OrganizationFilterInput

Allow retrieving a specific Organization

**Fields:**
- `organizationId`: `String!`

#### PinterestPostMetadataInput

Pinterest post metadata

**Fields:**
- `boardServiceId`: `String` - The board ID of the Pin, can be obtained when fetching the channel details with the following query:
```
query GetChannelWithSubprofiles {
  channel(input: { id: "[CHANNEL_ID_HERE]" }) {
    metadata {
      ... on PinterestMetadata {
        boards {
          serviceId
        }
      }
    }
  }
}
```
Required on create; optional on edit (omitted preserves existing board).
- `title`: `String` - The title of the Pin
- `url`: `String` - The Pin destination link

#### PostInput

Input for the post query

**Fields:**
- `id`: `PostId!` - The ID of the post to be retrieved

#### PostInputMetaData

Metadata of the post which differs based on the social network/service

**Fields:**
- `bluesky`: `BlueskyPostMetadataInput` - Metadata for Bluesky post
- `facebook`: `FacebookPostMetadataInput` - Metadata for Facebook post
- `google`: `GoogleBusinessPostMetadataInput` - Metadata for Google Business Profile post
- `instagram`: `InstagramPostMetadataInput` - Metadata for Instagram post
- `linkedin`: `LinkedInPostMetadataInput` - Metadata for LinkedIn post
- `mastodon`: `MastodonPostMetadataInput` - Metadata for Mastodon post
- `pinterest`: `PinterestPostMetadataInput` - Metadata for Pinterest post
- `threads`: `ThreadsPostMetadataInput` - Metadata for Threads post
- `tiktok`: `TikTokPostMetadataInput` - Metadata for TikTok post
- `twitter`: `TwitterPostMetadataInput` - Metadata for Twitter post
- `youtube`: `YoutubePostMetadataInput` - Metadata for Youtube post

#### PostsFiltersInput

Filter to apply to the posts query

**Fields:**
- `channelIds`: `[ChannelId!]` - When set, it will filter posts by channel
- `dueAt`: `DateTimeComparator` - When set, it will filter posts by their scheduled posting date
- `dueAtPresence`: `DateTimePresence` - When set, it will filter posts by whether their scheduled posting date exists.
`absent` cannot be combined with `dueAt`, because absent dates cannot also match a date range.
- `endDate`: `DateTime` - When set, it will return posts with createdAt or dueAt date before endDate
- `startDate`: `DateTime` - When set, it will return posts with createdAt or dueAt date after startDate
- `status`: `[PostStatus!]` - When set, it will filter posts by status
- `tagIds`: `[TagId!]` - When set, it will filter posts by tag
- `tags`: `TagComparator` - Filter posts by tags. Supports specific tags, untagged posts, or union of both.
- `createdAt`: `DateTimeComparator` - When set, it will filter posts by the date they were created

#### PostsInput

Input for the posts query

**Fields:**
- `filter`: `PostsFiltersInput` - The filters to apply to the posts query
- `organizationId`: `OrganizationId!` - The Organization id to fetch posts for
- `sort`: `[PostSortInput!]` - The sort to apply to the posts results

#### PostSortInput

Sort order of post results. List multiple to create tie-breaking order.

**Fields:**
- `direction`: `SortDirection!` - The direction to sort by.
- `field`: `PostSortableKey!` - The field to sort by.

#### PostTemplateInput

Input for fetching a single post template by ID.

**Fields:**
- `id`: `PostTemplateId!` - The unique identifier of the template to fetch.

#### PostTemplatesFilterInput

Filters for the template library. Visibility narrows the actor-scoped
result set; it can never widen it.

**Fields:**
- `visibility`: `PostTemplateVisibility` - Narrow the result to a single visibility scope. Omit to receive the
union of: public templates, internal templates from the supplied
organization, and private templates from the actor's account.

#### PostTemplatesInput

Input for fetching templates accessible to the current actor for the
template library. A caller can never reach templates outside their own
scope — the server pins `private` to the actor's account and `internal`
to the supplied `organizationId` regardless of input.

**Fields:**
- `filter`: `PostTemplatesFilterInput` - Optional filters to narrow the template list.
- `organizationId`: `OrganizationId!` - Organization to scope `internal`-visibility templates to. The caller
must be a member of this organization.

#### RetweetMetadataInput

Information about the initial Tweet that was retweeted

**Fields:**
- `id`: `String!` - Retweet ID
- `comment`: `String` - Optional user comment shown above the embedded retweet

#### TagComparator

Comparator for filtering by tags

**Fields:**
- `in`: `[TagId!]!` - Include results that have any of the specified tags (union/OR).
- `isEmpty`: `Boolean!` (default: false) - When true, include results that have no tags assigned.
Can be combined with 'in' for union filtering.
Defaults to false if not specified.

#### TagInput

Input type for tag information used in idea creation

**Fields:**
- `id`: `ID!`
- `name`: `String!`
- `color`: `String!`

#### ThreadedPostInput

A post authored by the user which is posted to a thread.
This is commonly used for long-format twitter and meta threads posts to
allow authored content to span multiple threads.
Threads are represented as a list of replies, each replying to the previous one.
The first item in the list is the root post of the thread and should match the
top-level `text` on the post input; the remaining items are replies.

**Fields:**
- `assets`: `[AssetInput!]!` (default: []) - Ordered list of assets on this threaded post
- `text`: `String` - The text body content of the threaded post

#### ThreadsPostMetadataInput

Threads post metadata

**Fields:**
- `linkAttachment`: `LinkAttachmentInput` - Link attachment. Mutually exclusive with a non-empty `assets` array — input providing both is rejected.
- `locationId`: `String` - LocationId associated with the post
- `locationName`: `String` - Location name associated with the post
- `thread`: `[ThreadedPostInput!]` - The ordered list of posts that make up the thread (not paginated). This
array is the source of truth for what gets published: every post in the
thread, including the root post, must be provided here. Posts are
published in order, each replying to the previous one. The first item is
the root post and should match the top-level `text` on the post input.
- `topic`: `String` - Topic associated with the post
- `type`: `PostType` - The type of the post

#### TikTokPostMetadataInput

TikTok post metadata

**Fields:**
- `isAiGenerated`: `Boolean` - Whether the post discloses AI-generated content (TikTok video only)
- `title`: `String` - The title of the TikTok post (for photo posts)

#### TwitterPostMetadataInput

Twitter post metadata

**Fields:**
- `isAiGenerated`: `Boolean` - Whether the post discloses AI-generated content (original tweets only, never retweets)
- `retweet`: `RetweetMetadataInput` - The details of the tweet being retweeted
- `thread`: `[ThreadedPostInput!]` - The ordered list of posts that make up the thread (not paginated). This
array is the source of truth for what gets published: every post in the
thread, including the root post, must be provided here. Posts are
published in order, each replying to the previous one. The first item is
the root post and should match the top-level `text` on the post input.

#### UpdatePostTemplateInput

Input for an end user updating a post template. Buffer-curated
taxonomy fields are ignored; setting them is only available to
official Buffer clients.

**Fields:**
- `id`: `PostTemplateId!` - The ID of the template to update.
- `body`: `String` - The main content body of the template, may contain {{placeholders}}.
- `description`: `String` - A short user-facing description of the template.
- `emoji`: `String` - The emoji associated with the template.
- `title`: `String` - The title of the template.
- `visibility`: `PostTemplateVisibility` - `public` is rejected — it is only available to official Buffer
clients.

#### UserTagInput

User tag in the image

**Fields:**
- `handle`: `String!` - The handle (username) of the account to tag, without the leading @.
- `x`: `Float!` - Horizontal position of the tag as a normalized decimal float between 0.0 and 1.0 - the fraction of the image width from the left edge (0.5 is the horizontal center). Pass a number, not a string, and do not use pixel coordinates; to convert, divide the pixel X by the image width.
- `y`: `Float!` - Vertical position of the tag as a normalized decimal float between 0.0 and 1.0 - the fraction of the image height from the top edge (0.5 is the vertical center). Pass a number, not a string, and do not use pixel coordinates; to convert, divide the pixel Y by the image height.

#### VideoAssetInput

Video asset

**Fields:**
- `metadata`: `VideoMetadataInput` - Video specific metadata
- `thumbnailUrl`: `String` - Do not use: social networks do not accept custom video thumbnail images, and
the API rejects video assets that set this field. To choose the video
thumbnail, set `metadata.thumbnailOffset` to select a frame from the video
(supported for Instagram, TikTok, and Pinterest only).
- `url`: `String!` - URL to the file source

#### VideoMetadataInput

Video metadata

**Fields:**
- `thumbnailOffset`: `Int` - Offset of the thumbnail chosen for the video, in ms
- `title`: `String` - Video title

#### YoutubePostMetadataInput

Youtube post metadata

**Fields:**
- `categoryId`: `String` - Youtube Category ID, one ID of this list:
ID: 1 -> Film & Animation
ID: 2 -> Autos & Vehicles
ID: 10 -> Music
ID: 15 -> Pets & Animals
ID: 17 -> Sports
ID: 19 -> Travel & Events
ID: 20 -> Gaming
ID: 22 -> People & Blogs
ID: 23 -> Comedy
ID: 24 -> Entertainment
ID: 25 -> News & Politics
ID: 26 -> Howto & Style
ID: 27 -> Education
ID: 28 -> Science & Technology
ID: 29 -> Nonprofits & Activism

Required on create; optional on edit (omitted preserves existing value).
- `embeddable`: `Boolean` - Indicates whether the video allows embedding (default: true)
- `isAiGenerated`: `Boolean` - Whether the post discloses AI-generated content
- `license`: `YoutubeLicense` - Video license (default: youtube)
- `madeForKids`: `Boolean` - Indicates whether the video is suitable for kids (default: false)
- `notifySubscribers`: `Boolean` - Indicates whether to notify subscribers on publish video (default: true)
- `privacy`: `YoutubePrivacy` - Privacy setting for post (default: public)
- `title`: `String` - Title of the Youtube post. Required on create; optional on edit (omitted preserves existing value).

## Interfaces

#### Asset

Asset interface with common fields

**Fields:**
- `id`: `ID` - The ID of the asset in the database
- `mimeType`: `String!` - The MIME type of the asset
- `source`: `String!` - URL to the file source
- `thumbnail`: `String!` - URL to the static thumbnail of the asset
- `type`: `AssetType!` - The type of the asset

#### CommonPostMetadata

Common properties for all post metadata types

**Fields:**
- `annotations`: `[Annotation!]!` - Annotations representing entities in the text
- `type`: `PostType!` - The channel-specific type of the post, eg, post, story, reel for Instagram

#### MutationError

Base Mutation Error type

**Fields:**
- `message`: `String!` - Error message

#### MutationSuccess

Base Mutation Success type
Used when we have a success response with no data, we return this type with empty string

**Fields:**
- `_empty`: `String!` - The value is alwaus an empty string ''
Note: GraphQL doesn't allow types with no fields, so we have to add this field

#### ScrapedLink

Link data for link preview

**Fields:**
- `text`: `String!` - Description for the scraped link
- `thumbnails`: `[String!]!` - Thumbnails of media available in the link
- `url`: `String!` - URL that the link asset has been built from

#### ThreadedPostMetadata

Common properties for all posts that support threaded replies.
See ThreadedPost for more details.

**Fields:**
- `thread`: `[ThreadedPost!]!` - The list of threaded posts (not paginated)
- `threadCount`: `Int!` - The number of threaded posts

## Unions

#### ChannelMetadata

Metadata or settings about the channel depending on the service type

**Possible types:** InstagramMetadata | TiktokMetadata | YoutubeMetadata | PinterestMetadata | MastodonMetadata | BlueskyMetadata | GoogleBusinessMetadata | FacebookMetadata | TwitterMetadata | LinkedInMetadata

#### CreateIdeaPayload

createIdea response (including errors)

**Possible types:** Idea | IdeaResponse | InvalidInputError | UnauthorizedError | UnexpectedError | LimitReachedError

#### CreatePostTemplatePayload

Result of an end user creating a post template.

**Possible types:** CreatePostTemplateSuccess | VoidMutationError

#### DeletePostPayload

All possible response types for the deletePost mutation.

**Possible types:** DeletePostSuccess | VoidMutationError

#### DeletePostTemplatePayload

Result of an end user deleting a post template.

**Possible types:** EmptySuccess | VoidMutationError

#### GoogleBusinessPostDetails

GoogleBusiness Metadata details

**Possible types:** GoogleBusinessWhatsNewMetaData | GoogleBusinessOfferMetaData | GoogleBusinessEventMetaData

#### MovePostInQueuePayload

All possible response types that can be returned by movePostInQueue mutation.

**Possible types:** PostActionSuccess | VoidMutationError

#### PostActionPayload

Create post's request response payload.

**Possible types:** PostActionSuccess | NotFoundError | UnauthorizedError | UnexpectedError | RestProxyError | LimitReachedError | InvalidInputError

#### PostMetadata

Post metadata union type. Contains all possible types of post metadata.

**Possible types:** InstagramPostMetadata | FacebookPostMetadata | LinkedInPostMetadata | TwitterPostMetadata | PinterestPostMetadata | GoogleBusinessPostMetadata | YoutubePostMetadata | MastodonPostMetadata | TiktokPostMetadata | ThreadsPostMetadata | BlueskyPostMetadata

#### UpdatePostTemplatePayload

Result of an end user updating a post template.

**Possible types:** UpdatePostTemplateSuccess | VoidMutationError

## Enums

#### AnnotationType

List of possible types for an annotation

**Values:**
- `annotation`
- `cashtag`
- `hashtag`
- `mention`
- `url`

#### AssetType

Asset types

**Values:**
- `document`
- `image`
- `video`

#### ChannelAction

List of possible actions that can be performed on a Channel

**Values:**
- `backfillChannel`
- `manageCapabilities`
- `manageComments`
- `manageIntegrations`
- `managePostingSchedule`
- `manageUpdates`
- `publishStartPage`
- `readUpdates`
- `reconnectChannel`
- `removeChannel`
- `viewCapabilities`
- `viewChannel`
- `viewComments`
- `viewInsights`
- `viewPublish`
- `viewUpdates` *(Deprecated: Renamed to `viewPublish`. Still emitted for backward-compat with clients that gate on `viewUpdates`; use `viewPublish` to gate the Publish UI or `readUpdates` to gate reading post data. Will be removed once no client reads it.)*

#### ChannelType

Channel is a representation of a social media account or page that can be connected to Buffer.

**Values:**
- `account`
- `business`
- `channel`
- `group`
- `page`
- `profile`

#### ConnectedAppCategory

The category of a connected app.

**Values:**
- `mcp` - An MCP client connection.

#### DateTimePresence

Presence filter for nullable date fields.
When filtering the same field, absent dates cannot also match a date comparator range.

**Values:**
- `absent` - Include results where the date field is missing or null
- `present` - Include results where the date field exists and is not null

#### DayOfWeek

Day of the week.

**Values:**
- `fri`
- `mon`
- `sat`
- `sun`
- `thu`
- `tue`
- `wed`

#### GoogleBusinessPostActionType

List of possible types for GBP cta

**Values:**
- `book`
- `call`
- `learn_more`
- `none`
- `order`
- `shop`
- `signup`

#### IdeaGroupMembership

Named buckets for filtering ideas by their group membership.

**Values:**
- `grouped` - Only ideas that are assigned to a group.
- `ungrouped` - Only ideas that are not assigned to any group.

#### MediaType

The type of media attached to a post

**Values:**
- `image`
- `gif`
- `video`
- `link`
- `document`
- `unsupported`

#### NoteAction

List of possible actions that can be performed on a note

**Values:**
- `deleteNote` - The user can delete the note.
- `updateNote` - The user can update the note.

#### NoteType

The type of a note.

**Values:**
- `aiGenerated` - A note that was generated by our AI system.
- `bufferGenerated` - A note that was generated by our internal system. Can be used for approval flows notifications or other automated processes.
- `userGenerated` - A note that was manually written by a user.

#### NotificationStatus

List of possible statuses for a notification

**Values:**
- `markedAsPublished`
- `notified`

#### OrganizationAction

List of possible actions that can be performed on a Organization

**Values:**
- `createPostGroup`
- `createPostTemplate`
- `edit`
- `manageAllNotes`
- `manageBilling`
- `manageChannelGroups`
- `manageChannels`
- `manageIntegrations`
- `manageSecurityTFAEnforcement`
- `manageTeamMembers`
- `manageTrustedPartnerStatus`
- `manageUploads`
- `publishStartPages`
- `readUploads`
- `receiveOrganizationOwnership`
- `transferOwnership`
- `view`

#### PostAction

List of possible actions that can be performed on a Post

**Values:**
- `addPostNote`
- `addPostToQueue`
- `approvePost`
- `cancelPostRecurrence`
- `copyPostLink`
- `createPostRecurrence`
- `deletePost`
- `duplicatePost`
- `editPostRecurrence`
- `movePostToDraft`
- `publishPostNext`
- `publishPostNow`
- `rejectPost`
- `removePostScheduledTime`
- `requestPostApproval`
- `revertPostApprovalRequest`
- `sharePostLink`
- `updatePost`
- `updatePostSchedule`
- `updatePostTags`
- `updateShopGridLink`
- `viewPost`

#### PostApprovalChange

A change to a post's approval state, for a post that is already a draft.

**Values:**
- `request` - Submit the draft for approval.
- `revert` - Withdraw a pending approval request, returning the post to a plain draft.

#### PostingGoalStatus

PostingGoalStatus is used to track the status of a posting goal.

**Values:**
- `AtRisk`
- `Hit`
- `OnTrack`

#### PostMetricType

List of possible metrics available for a Post.

Values fall into three groups:
- **Cross-network normalized** (reactions, comments, shares, reposts, reach, impressions, views, clicks, engagementRate): Used wherever a concept maps cleanly across networks. Per-network adapters normalize their native names (e.g., Instagram `likes` → `reactions`, Twitter `retweets` → `reposts`).
- **Network-specific** (saves, follows, quotes, viewers, totalTimeWatched, likes): Real metrics that don't have a cross-network equivalent. `likes` is intentionally distinct from `reactions` on Facebook — Facebook's Graph API surfaces them separately.
- **Aggregation-only** (postCount): Meaningful only on aggregate endpoints; never emitted per-post.

Deprecated values are pre-normalization legacy or tied to features being removed. They're kept in the enum for backwards compatibility until clients migrate.

**Values:**
- `clicks` - How many times people clicked on your post.
- `comments` - The count of comments and replies on your post. Unified across networks (Threads `replies` maps here).
- `engagementRate` - The percentage of people who interacted with your post compared to how many saw it. Unit: percentage.
- `favorites` - Deprecated: not emitted by any per-network definition. Use `reactions` instead — Twitter and Mastodon favorites normalize into `reactions`. Will be removed on 2026-12-01. *(Deprecated: Not emitted by any per-network definition; use `reactions` instead. Will be removed on 2026-12-01.)*
- `follows` - The number of new followers gained from this post (Instagram).
- `impressions` - How many times your post was shown on screen. May include multiple views by the same person — useful for spotting how often the content gets surfaced.
- `likes` - The Like-reaction subcount on Facebook. Distinct from `reactions` (which is the total of all reaction types — Like, Love, Care, Haha, Wow, Sad, Angry); Facebook's Graph API reports them separately and we mirror that.
- `link_clicks` - Deprecated: StartPage link-clicks metric. StartPage is being deprecated as a product. Will be removed on 2026-12-01. *(Deprecated: StartPage is being deprecated as a product. Will be removed on 2026-12-01.)*
- `other` - Deprecated catch-all from pre-normalization. Never emitted. Will be removed on 2026-12-01. *(Deprecated: Catch-all from pre-normalization; never emitted. Will be removed on 2026-12-01.)*
- `postCount` - The count of posts included in an aggregated response. Only meaningful on aggregate endpoints — never emitted per-post.
- `quotes` - How many times your post was quoted (Threads).
- `reach` - The number of unique people who saw your post.
- `reactions` - How many people reacted to your post. Unified across networks: Instagram/Twitter `likes`, Mastodon `favorites`, etc. all map to this value.
- `reblogs` - Deprecated: not emitted by any per-network definition. Use `reposts` instead — Mastodon reblogs normalize into `reposts`. Will be removed on 2026-12-01. *(Deprecated: Not emitted by any per-network definition; use `reposts` instead. Will be removed on 2026-12-01.)*
- `repins` - Deprecated: not emitted by any per-network definition. Pre-normalization legacy from the Pinterest era. Will be removed on 2026-12-01. *(Deprecated: Not emitted by any per-network definition. Will be removed on 2026-12-01.)*
- `replies` - Deprecated: not emitted by any per-network definition. Use `comments` instead — replies are normalized into `comments` on the networks that distinguish them (Threads). Will be removed on 2026-12-01. *(Deprecated: Not emitted by any per-network definition; use `comments` instead. Will be removed on 2026-12-01.)*
- `reposts` - How many times your post was reposted by others. Twitter `retweets`, Mastodon `reblogs`, Threads `reposts` all normalize to this value.
- `retweets` - Deprecated: not emitted by any per-network definition. Use `reposts` instead — Twitter retweets normalize into `reposts`. Will be removed on 2026-12-01. *(Deprecated: Not emitted by any per-network definition; use `reposts` instead. Will be removed on 2026-12-01.)*
- `saves` - How many times people saved your post (Instagram, Pinterest). A strong signal that the content is worth revisiting.
- `shares` - How many times your post was shared or forwarded by others.
- `totalTimeWatched` - Total time watched, in minutes, for video-style posts (LinkedIn).
- `viewers` - Unique viewer count for video-style posts (LinkedIn).
- `views` - How many times your post was viewed. Used for video-style posts and on networks that report views distinctly from impressions.

#### PostMetricUnit

The unit representing the value of a PostMetric.

**Values:**
- `count` - An integer count (e.g. number of reactions, impressions, reach).
- `percentage` - A percentage value between 0 and 100 (e.g. engagement rate).

#### PostSortableKey

Key of collection to use for sorting

**Values:**
- `dueAt` - Sort by the post's dueAt field.
Due at is the date when the post is scheduled to be published.
- `createdAt` - Sort by the post's createdAt field.
Created at is the date when the post was created.

#### PostStatus

List of possible statuses for a Post

**Values:**
- `draft`
- `error`
- `needs_approval`
- `scheduled`
- `sending`
- `sent`

#### PostTemplateVisibility

The visibility level of a post template. `public` is reserved for
Buffer-curated templates; setting it is only available to official
Buffer clients.

**Status:** 🧪 Preview

**Values:**
- `internal` - All members of the template's organization can access this template.
- `private` - Only the creator can access this template.
- `public` - Anyone across all organizations can access this template. Reserved
for Buffer-curated templates — `createPostTemplate` and
`updatePostTemplate` reject this value; setting it is only available
to official Buffer clients. It appears in read results.

#### PostType

List of possible types for a Post. Some services may have different types (e.g., Instagram has story, reel, post but Twitter has only post)

**Values:**
- `carousel`
- `event`
- `ghost_post`
- `offer`
- `post`
- `reel`
- `short`
- `story`
- `thread`
- `whats_new`

#### PostTypeFacebook

List of specific post types available for Facebook

**Values:**
- `post`
- `reel`
- `story`

#### PostTypeGoogleBusiness

List of specific post types available for Google Business profiles

**Values:**
- `event`
- `offer`
- `whats_new`

#### PostVia

List of possible ways to create a Post

**Values:**
- `api`
- `buffer`
- `network`

#### Product

Buffer products, buffer is used as all products

**Values:**
- `analyze`
- `buffer`
- `comments`
- `engage`
- `publish`
- `startPage`

#### QueuePosition

Target position within a channel's queue that a post can be moved to.

**Status:** ⚠️ Experimental

**Values:**
- `bottom` - Move the post to the bottom of the queue, taking the last slot.
- `top` - Move the post to the top of the queue, taking the next available slot.

#### ScheduleOption

**Values:**
- `Queue`
- `Prioritize`
- `FixedTime`
- `Now`

#### SchedulingType

Indicates whether the post was scheduled for notification publishing or automatic publishing

**Values:**
- `automatic` - Buffer's publishing workers send the post, with nobody having to act
- `notification` - Buffer reminds someone to publish the post by hand

#### Service

The list of services that can be authorized.

**Values:**
- `bluesky`
- `facebook`
- `googlebusiness`
- `instagram`
- `linkedin`
- `mastodon`
- `pinterest`
- `startPage`
- `threads`
- `tiktok`
- `twitter`
- `youtube`

#### ShareMode

How the post is being scheduled.

**Values:**
- `addToQueue`
- `customScheduled`
- `shareNext`
- `shareNow`

#### SortDirection

Direction to sort the results by.

**Values:**
- `asc` - Sort records in ascending order.
- `desc` - Sort records in descending order.

#### TagColorName

Stable Buffer identifiers for the supported tag color palette.
Clients map these values to platform- and theme-specific presentation.
Values ending in `Light` are lighter palette variants, not UI theme modes.

**Values:**
- `blue` - The standard blue palette color.
- `blueLight` - The lighter blue palette color.
- `gray` - The standard gray palette color.
- `grayLight` - The lighter gray palette color.
- `green` - The standard green palette color.
- `greenLight` - The lighter green palette color.
- `orange` - The standard orange palette color.
- `orangeLight` - The lighter orange palette color.
- `pink` - The standard pink palette color.
- `pinkLight` - The lighter pink palette color.
- `purple` - The standard purple palette color.
- `purpleLight` - The lighter purple palette color.
- `red` - The standard red palette color.
- `redLight` - The lighter red palette color.
- `teal` - The standard teal palette color.
- `tealLight` - The lighter teal palette color.
- `yellow` - The standard yellow palette color.
- `yellowLight` - The lighter yellow palette color.

#### YoutubeLicense

List of license types

**Values:**
- `creativeCommon`
- `youtube`

#### YoutubePrivacy

List of privacy types

**Values:**
- `private`
- `public`
- `unlisted`

## Scalars

#### AccountId

The `AccountId` scalar represents the MongoDB ObjectId of a Buffer Account

#### ChannelId

The `ChannelId` scalar represents the MongoDB ObjectId of a Buffer Channel

#### DateTime

The `DateTime` scalar represents a date and time following the ISO 8601 standard.

#### DraftId

The `DraftId` scalar represents the MongoDB ObjectId of a Buffer Draft

#### Email

The `Email` scalar represents a valid, normalized email address.
Input is trimmed and lowercased before validation.

#### IdeaId

The `IdeaId` scalar represents the MongoDB ObjectId of a Buffer Idea

#### InvitationId

The `InvitationId` scalar represents the MongoDB ObjectId of a pending team invitation

#### NoteId

The `NoteId` scalar represents the MongoDB ObjectId of a Buffer Note

#### OrganizationId

The `OrganizationId` scalar represents the MongoDB ObjectId of a Buffer Organization

#### PostGroupId

The `PostGroupId` scalar represents the MongoDB ObjectId of a Buffer Post Group.

#### PostId

The `PostId` scalar represents the MongoDB ObjectId of a Buffer Post

#### PostTemplateId

The `PostTemplateId` scalar represents the MongoDB ObjectId of a Post Template.

#### TagId

The `TagId` scalar represents the MongoDB ObjectId of a Buffer Tag

#### Uuid

The `Uuid` scalar represents an RFC 4122 v4 UUID,
e.g. `550e8400-e29b-41d4-a716-446655440000`.
