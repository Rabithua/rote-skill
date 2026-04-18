# Rote command map

Auth config lives at `~/.rote-toolkit/config.json`.
Public explore-note reads do not require auth.

## Setup

```bash
rote config
```

## CLI quick commands

```bash
rote add "note content"
rote add "note content" --title "Daily Note" -t "journal,daily"
rote add "note content" --public --pin
rote add "note content" --article-id "<articleId>"
rote article add "article content"
rote reaction add <roteid> like
rote reaction remove <roteid> like
rote profile get
rote profile update --nickname "New Name" --description "Bio"
rote permissions
rote explore --limit 20 --skip 0
rote search "keyword" --limit 20 --skip 20
rote search "keyword" --archived -t "tag1,tag2"
rote list --limit 10 --skip 0 --archived -t "tag1,tag2"
rote articles --limit 20 --skip 0 -k "keyword"
rote tags
rote heatmap --start 2024-01-01 --end 2024-12-31
rote stats
rote settings get
rote settings update --allow-explore true
rote mcp
```

## High-level CLI workflows

Create a public pinned note with tags:

```bash
rote add "Weekly launch recap" --title "Launch Recap" -t "weekly,launch" --public --pin
```

Create an article, then attach future notes to it:

```bash
rote article add "# Project Digest"
rote add "Digest summary" --article-id "<articleId>" -t "digest"
```

Review recent notes in a topic area:

```bash
rote search "MCP" --limit 10 -t "ai,notes"
rote list --limit 20 -t "ai"
```

Check account state before sensitive operations:

```bash
rote permissions
rote profile get
```

Browse public explore notes without auth:

```bash
rote explore --limit 10
```

## SDK surface

Import from the package root:

```ts
import { RoteClient } from "rote-toolkit";
```

Primary methods:

- `createNote`
- `updateNote`
- `deleteNote`
- `searchNotes`
- `listNotes`
- `exploreNotes`
- `createArticle`
- `listArticles`
- `getArticleByNoteId`
- `batchGetNotes`
- `addReaction`
- `removeReaction`
- `getProfile`
- `updateProfile`
- `getPermissions`
- `getTags`
- `getHeatmap`
- `getStatistics`
- `getSettings`
- `updateSettings`
- `batchDeleteAttachments`
- `updateAttachmentsSortOrder`

## SDK patterns

Create a note:

```ts
const client = new RoteClient();
const note = await client.createNote({
  content: "Today I learned MCP",
  title: "Learning Log",
  tags: ["journal", "ai"],
  isPublic: false,
  pin: false,
});
```

Update visibility, tags, archive state, or bound article:

```ts
await client.updateNote({
  noteId: "<noteId>",
  title: "Revised title",
  tags: ["knowledge", "rote"],
  isPublic: true,
  pin: true,
  archived: false,
  articleId: "<articleId>",
});
```

Search or list with pagination:

```ts
const searchResults = await client.searchNotes({
  keyword: "MCP",
  limit: 20,
  skip: 0,
  archived: false,
  tag: ["ai", "notes"],
});

const recent = await client.listNotes({
  limit: 20,
  skip: 20,
  archived: true,
  tag: ["archive"],
});

const explore = await client.exploreNotes({
  limit: 20,
  skip: 0,
});
```

Profile and permission checks:

```ts
const profile = await client.getProfile();
const permissions = await client.getPermissions();

await client.updateProfile({
  nickname: "New Name",
  description: "Builder and note taker",
});
```

Reactions:

```ts
await client.addReaction({ roteid: "<noteId>", type: "like" });
await client.removeReaction({ roteid: "<noteId>", type: "like" });
```

The `metadata` field is an optional JSON object stored alongside the reaction.
Its primary use is recording the **source channel** via a `source` key:

| Channel | `metadata.source` | Injected by |
|---------|-------------------|-------------|
| Web UI  | `"web"`           | frontend    |
| CLI     | `"cli"`           | cli.ts      |
| MCP     | `"mcp"`           | mcp.ts      |
| SDK     | caller decides    | user code   |

CLI and MCP automatically inject `{ source: "cli" }` / `{ source: "mcp" }`.
SDK callers can pass any extra key-value pairs alongside `source`.

Extended API operations:

```ts
// List articles
const articles = await client.listArticles({ limit: 20, skip: 0, keyword: "digest" });

// Batch get notes by IDs (max 100)
const notes = await client.batchGetNotes({ ids: ["id1", "id2", "id3"] });

// Get tag statistics
const tags = await client.getTags();

// Get activity heatmap
const heatmap = await client.getHeatmap({
  startDate: "2024-01-01",
  endDate: "2024-12-31",
});

// Get statistics
const stats = await client.getStatistics();

// Get and update settings
const settings = await client.getSettings();
await client.updateSettings({ allowExplore: true });

// Batch delete attachments (max 100)
const result = await client.batchDeleteAttachments({ ids: ["attachmentId1", "attachmentId2"] });

// Update attachment sort order
await client.updateAttachmentsSortOrder({
  noteId: "<noteId>",
  attachmentIds: ["att1", "att2", "att3"],
});
```

## High-level operations

Use SDK or MCP for these composed tasks:

- Search, inspect, then selectively update matching notes.
- Bulk-archive or bulk-unarchive note sets after filtering by keyword, tag, or pagination window.
- Migrate notes from private to public in a controlled batch.
- Re-tag old notes by searching a topic and updating each note with normalized tags.
- Attach many notes to a shared `articleId` after creating the article once.
- Build account diagnostics by combining `getProfile` and `getPermissions`.
- Add or remove reactions across a search result set.
- Pull public explore notes for discovery or inspiration flows before authenticated operations.

Typical batch update loop:

```ts
const client = new RoteClient();
const notes = await client.searchNotes({ keyword: "draft", limit: 50 });

for (const note of notes) {
  await client.updateNote({
    noteId: note.id,
    archived: true,
    tags: ["draft", "archived"],
  });
}
```

## Update strategy

- Treat `updateNote` as a partial update, but only send fields you intend to change.
- When normalizing tags, decide whether the task means replace or merge. Do not silently drop existing tags unless the user asked for replacement.
- When changing visibility, send only `isPublic` and any explicitly requested fields.
- When archiving or unarchiving, avoid rewriting content unless the task also asks for content changes.
- When rebinding notes to an `articleId`, preserve title, tags, pin state, and visibility unless the user asked to alter them.

Safe merge pattern for tags:

```ts
const mergedTags = Array.from(
  new Set([...(note.tags ?? []), "knowledge", "reviewed"]),
);

await client.updateNote({
  noteId: note.id,
  tags: mergedTags,
});
```

## Batch operation guardrails

- Discover targets first with `searchNotes` or `listNotes`; do not update blind IDs unless the user already supplied them.
- Paginate through large result sets with `limit` and `skip` instead of assuming one page is complete.
- Keep batch sizes conservative, such as 20 to 50 notes per pass.
- If the operation is destructive or high-impact, summarize the target set before executing.
- For mixed-result batches, continue collecting successes and failures instead of aborting on the first failure unless the task is explicitly all-or-nothing.
- After batch updates, report counts for scanned, changed, skipped, and failed notes.

Paginated scan pattern:

```ts
const client = new RoteClient();
const pageSize = 20;

for (let skip = 0; ; skip += pageSize) {
  const notes = await client.searchNotes({
    keyword: "draft",
    limit: pageSize,
    skip,
  });

  if (notes.length === 0) break;

  for (const note of notes) {
    await client.updateNote({
      noteId: note.id,
      archived: true,
    });
  }
}
```

## Failure handling

- If config is missing, run `rote config` before retrying.
- If the task is only to read public explore notes, do not block on missing config.
- If a task references a note loosely, resolve it with `search` or `list` before `update` or `delete`.
- If the API key may be restricted, run `rote permissions` or `getPermissions` before write operations.
- If `updateNote` would send no changed fields, stop and ask for the intended mutation instead of issuing a noop.
- If a batch search returns ambiguous matches, summarize candidates and narrow the target set before mutating.
- If the API returns a request failure, surface the server message directly because `RoteClient` already preserves it.

## Playbooks

### Bulk archive playbook

Use when the user wants to archive or unarchive many notes selected by keyword, tags, or a review window.

Steps:

1. Discover candidate notes with `searchNotes` or `listNotes`.
2. Summarize the candidate count and selection rule if the change is high-impact.
3. Update only `archived` unless the task explicitly asks for more.
4. Report scanned, changed, skipped, and failed counts.

```ts
const client = new RoteClient();
const notes = await client.searchNotes({
  keyword: "draft",
  limit: 50,
  archived: false,
});

let changed = 0;
for (const note of notes) {
  await client.updateNote({
    noteId: note.id,
    archived: true,
  });
  changed += 1;
}
```

### Retagging playbook

Use when the user wants to normalize tags, add a new taxonomy, or remove obsolete labels across many notes.

Steps:

1. Search or list the target note set.
2. Decide whether tags should be merged or replaced.
3. Preserve unrelated existing tags unless the user explicitly wants replacement.
4. Update only notes whose final tag set actually changes.

```ts
const client = new RoteClient();
const notes = await client.searchNotes({
  keyword: "mcp",
  limit: 50,
});

for (const note of notes) {
  const nextTags = Array.from(
    new Set([...(note.tags ?? []), "ai", "knowledge"]),
  );

  const unchanged =
    JSON.stringify([...(note.tags ?? [])].sort()) ===
    JSON.stringify([...nextTags].sort());
  if (unchanged) continue;

  await client.updateNote({
    noteId: note.id,
    tags: nextTags,
  });
}
```

### Article binding playbook

Use when the user wants a group of notes attached to one article record.

Steps:

1. Create the article once with `createArticle`.
2. Search or list the notes that should be bound.
3. Update each note with `articleId` only, unless the task asks for more changes.
4. Preserve tags, visibility, title, and pin state.

```ts
const client = new RoteClient();
const article = await client.createArticle({
  content: "# Project Digest",
});

const notes = await client.searchNotes({
  keyword: "digest",
  limit: 20,
});

for (const note of notes) {
  await client.updateNote({
    noteId: note.id,
    articleId: article.id,
  });
}
```

### Permission diagnostics playbook

Use when writes fail unexpectedly, when a new key is being verified, or before sensitive operations.

Steps:

1. Read the current profile with `getProfile`.
2. Read permissions with `getPermissions`.
3. Compare the intended operation against the available permission set.
4. If permissions are insufficient, stop and surface the gap instead of retrying blindly.

```ts
const client = new RoteClient();
const profile = await client.getProfile();
const permissions = await client.getPermissions();

console.log({
  username: profile.username,
  permissions: permissions.permissions,
});
```

### Search-then-update playbook

Use when the user describes notes by topic or content rather than by exact ID.

Steps:

1. Resolve likely matches with `searchNotes`.
2. If multiple candidates appear, summarize the top matches and narrow the target set.
3. Apply `updateNote` only after the target note IDs are known.
4. Report exactly which notes were changed.

```ts
const client = new RoteClient();
const matches = await client.searchNotes({
  keyword: "launch recap",
  limit: 10,
});

for (const note of matches) {
  await client.updateNote({
    noteId: note.id,
    pin: true,
  });
}
```

### Explore discovery playbook

Use when the user wants public notes from the explore page, trend sampling, or inspiration material without requiring account auth.

Steps:

1. Use `exploreNotes` or `rote explore` first.
2. Paginate with `limit` and `skip` for broader sampling.
3. Treat results as public discovery data, not as directly mutable targets.
4. Switch to authenticated search or update flows only if the user then wants operations on owned notes.

```ts
const client = new RoteClient();
const notes = await client.exploreNotes({
  limit: 20,
  skip: 0,
});
```

## MCP tools

Available tools:

- `rote_create_note`
- `rote_update_note`
- `rote_delete_note`
- `rote_create_article`
- `rote_list_articles`
- `rote_get_article_by_note`
- `rote_batch_get_notes`
- `rote_add_reaction`
- `rote_remove_reaction`
- `rote_get_profile`
- `rote_update_profile`
- `rote_get_permissions`
- `rote_search_notes`
- `rote_list_notes`
- `rote_explore_notes`
- `rote_get_tags`
- `rote_get_heatmap`
- `rote_get_statistics`
- `rote_get_settings`
- `rote_update_settings`
- `rote_batch_delete_attachments`
- `rote_update_attachments_sort`

## MCP usage guidance

- Use `rote_search_notes` or `rote_list_notes` first when the target note ID is unknown.
- Use `rote_explore_notes` for public note discovery when authentication is unnecessary or unavailable.
- Use `rote_update_note` for content edits, tag changes, visibility changes, pinning, archiving, or article rebinding.
- Use `rote_create_article` plus `rote_create_note`/`rote_update_note` when building a note collection under one article.
- Use `rote_get_permissions` before operations that may fail due to restricted keys.
