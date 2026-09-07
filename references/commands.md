# Rote command and tool reference

Sharing requires Rote Server 2.4.0 or later. Local/OpenKey workflows require `rote-toolkit` 0.6.0 or later.

## Authentication paths

### Rote Server HTTP MCP

Use an already-connected remote MCP when available. It authenticates through OAuth. Request only the scopes needed for the task; `notes:share` is not a default scope.

Never replace an OAuth failure with a Toolkit OpenKey operation without telling the user. An OAuth token, an OpenKey, and a note share token are different credentials.

### rote-toolkit

Toolkit CLI, SDK, and stdio MCP use the OpenKey stored in `~/.rote-toolkit/config.json`:

```bash
rote config
rote permissions
```

`SHAREROTE` is required for share status, creation, and revocation and is not selected for a newly created OpenKey by default.

## CLI

### Notes

```bash
rote add "note content"
rote add "note content" --title "Daily Note" -t "journal,daily"
rote add "note content" --public --pin
rote add "note content" --article-id "<articleId>"
rote get "<noteId>"
rote search "keyword" --limit 20 --skip 0
rote search "keyword" --archived -t "tag1,tag2"
rote list --limit 20 --skip 0 --archived -t "tag1,tag2"
rote explore --limit 20 --skip 0
```

Use the SDK or an MCP tool for note update/delete operations. When the note is described by content rather than ID, search first and narrow ambiguous matches before mutation.

### Articles

```bash
rote article add "# Article content"
rote article get "<articleId>"
rote article update "<articleId>" "# Revised content"
rote article delete "<articleId>"
rote articles --limit 20 --skip 0 -k "keyword"
```

### Share links

```bash
rote share status "<noteId>"
rote share create "<noteId>"
rote share revoke "<noteId>"
```

`share create` checks the server-configured frontend origin before creating a link. It does not fall back to an official domain. A status response is either `{ "active": false }` or contains `active`, `token`, `createdAt`, and `url`; `url` can be `null` when an existing token cannot be presented as a browser URL.

Run `share create` or `share revoke` only after an explicit user request. Do not paste the resulting bearer URL into command logs or repeat it in a completion summary.

### Attachments

```bash
rote attachment upload "<noteId>" ./photo.jpg ./clip.mp4
rote attachment delete "<attachmentId>"
```

The upload command accepts explicit image/video paths, obtains presigned URLs, uploads each file with PUT, and finalizes the records on the note. It does not scan directories or discover files implicitly. Live Photo pairs are available only through the SDK primitives described below.

### Other account operations

```bash
rote reaction add "<noteId>" like
rote reaction remove "<noteId>" like
rote profile get
rote profile update --nickname "New Name" --description "Bio"
rote tags
rote heatmap --start 2024-01-01 --end 2024-12-31
rote stats
rote settings get
rote settings update --allow-explore true
rote mcp
```

## SDK

```ts
import { RoteClient } from "rote-toolkit";

const client = new RoteClient();
```

### Method map

- Notes: `createNote`, `getNote`, `updateNote`, `deleteNote`, `searchNotes`, `listNotes`, `exploreNotes`, `batchGetNotes`
- Articles: `createArticle`, `getArticle`, `updateArticle`, `deleteArticle`, `listArticles`, `getArticleByNoteId`
- Shares: `getNoteShare`, `createNoteShare`, `revokeNoteShare`, `getNoteShareState`, `createResolvedNoteShare`, `resolveNoteShareUrl`
- Attachments: `presignAttachmentUploads`, `refreshAttachmentUploadReservation`, `finalizeAttachmentUploads`, `deleteAttachment`, `batchDeleteAttachments`, `updateAttachmentsSortOrder`
- Other: `addReaction`, `removeReaction`, `getProfile`, `updateProfile`, `getPermissions`, `getTags`, `getHeatmap`, `getStatistics`, `getSettings`, `updateSettings`

### Safe partial updates

```ts
const note = await client.getNote("<noteId>");
await client.updateNote({
  noteId: note.id,
  tags: Array.from(new Set([...(note.tags ?? []), "reviewed"])),
});

const article = await client.getArticle("<articleId>");
await client.updateArticle({
  articleId: article.id,
  content: "# Revised content",
});
```

Send only fields the user asked to change. Decide whether a tag operation means merge or replace; do not silently discard existing tags.

### Share lifecycle

```ts
const state = await client.getNoteShareState("<noteId>");

// Explicit user request required:
const share = await client.createResolvedNoteShare("<noteId>");

// Explicit user request required:
await client.revokeNoteShare("<noteId>");
```

`createResolvedNoteShare` validates the configured frontend origin before the share-record PUT. Use the raw `getNoteShare` method when an existing token must remain visible even if URL resolution is unavailable.

### Attachment primitives

```ts
const upload = await client.presignAttachmentUploads({
  files: [
    {
      filename: "photo.heic",
      contentType: "image/heic",
      size: 1234,
      mediaKind: "livePhoto",
      pairedVideo: {
        filename: "photo.mov",
        contentType: "video/quicktime",
        size: 5678,
      },
    },
  ],
});

// PUT bytes to upload.items[0].original.putUrl and
// upload.items[0].pairedVideo.putUrl. Refresh only if the reservation expires.

await client.finalizeAttachmentUploads({
  noteId: "<noteId>",
  attachments: [
    {
      uuid: upload.items[0].uuid,
      originalKey: upload.items[0].original.key,
      pairedVideoKey: upload.items[0].pairedVideo?.key,
      pairedVideoSize: 5678,
      pairedVideoMimetype: "video/quicktime",
      pairedVideoFilename: "photo.mov",
      size: 1234,
      mimetype: "image/heic",
      mediaKind: "livePhoto",
    },
  ],
});
```

The caller owns the actual PUT requests when using the SDK primitives. Do not log presigned URLs. If `reservationId` and `expiresAt` indicate expiry, call `refreshAttachmentUploadReservation(reservationId)` and use the returned URLs before PUT.

## OAuth HTTP MCP tools

These tools are exposed by Rote Server and use OAuth:

- Notes: `notes_create`, `notes_list`, `notes_search`, `notes_get`, `notes_batch_get`, `notes_update`, `notes_delete`
- Shares: `notes_share_get`, `notes_share_create`, `notes_share_revoke`
- Articles: `articles_create`, `articles_list`, `articles_get`, `articles_get_by_note`, `articles_update`, `articles_delete`
- Reactions: `reactions_add`, `reactions_remove`
- Account/data: `profile_get`, `profile_update`, `permissions_get`, `tags_get`, `heatmap_get`, `statistics_get`, `settings_get`, `settings_update`
- Attachments: `attachments_presign_upload`, `attachments_finalize_upload`, `attachments_sort`, `attachments_delete_one`, `attachments_delete_many`

Share tools appear only when the authorization includes `notes:share`. Creation validates the configured frontend origin before writing; status can return an existing token with `url: null`.

## Toolkit stdio MCP tools

These tools run locally and use the configured OpenKey:

- Notes: `rote_create_note`, `rote_get_note`, `rote_update_note`, `rote_delete_note`, `rote_search_notes`, `rote_list_notes`, `rote_explore_notes`, `rote_batch_get_notes`
- Shares: `rote_get_note_share`, `rote_create_note_share`, `rote_revoke_note_share`
- Articles: `rote_create_article`, `rote_get_article`, `rote_update_article`, `rote_delete_article`, `rote_list_articles`, `rote_get_article_by_note`
- Reactions: `rote_add_reaction`, `rote_remove_reaction`
- Account/data: `rote_get_profile`, `rote_update_profile`, `rote_get_permissions`, `rote_get_tags`, `rote_get_heatmap`, `rote_get_statistics`, `rote_get_settings`, `rote_update_settings`
- Attachments: `rote_delete_attachment`, `rote_batch_delete_attachments`, `rote_update_attachments_sort`

Toolkit stdio MCP intentionally exposes no local-file upload tool and accepts no local path for attachment uploads.

## Permissions and failure handling

- Use `permissions_get` for OAuth or `rote permissions` / `getPermissions` for OpenKey diagnostics.
- Do not retry a missing `notes:share` scope with OpenKey, or a missing `SHAREROTE` permission with OAuth, without an explicit user choice.
- If a target is ambiguous, return candidate IDs before mutating.
- For destructive batches, summarize the target set first and report scanned, changed, skipped, and failed counts.
- Surface server errors without embedding authorization headers, OpenKeys, share tokens, bearer URLs, or presigned URLs.
- Anonymous share reading remains `/v2/api/shares/:token`; a share token never authorizes Open API or MCP operations.
