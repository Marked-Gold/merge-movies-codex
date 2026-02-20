---
name: update-movie
description: Modify an existing merge.mov movie — add, edit, reorder, or remove scenes
---

# Update Movie

Modify an existing merge.mov movie using MCP tools. Add, edit, reorder, or remove scenes.

## Setup

The `merge-movies` MCP server must be connected. Authentication is handled via the `MERGE_MOVIES_API_KEY` environment variable passed through the MCP transport.

If the key is missing, tell the user to create one at [merge.mov Settings](https://merge.mov/settings).

## Workflow

### 1. Find the Target Movie

**By ID** (UUID-like string):
```
get_movie({ movieId: "<movie-id>" })
```

**By title search** — list movies and match:
```
list_movies({})
→ Returns array of { id, title, updatedAt }
```

**No input** — list movies and ask the user to pick.

### 2. Fetch Current State

Get the full movie with all scenes:

```
get_movie({ movieId: "<movie-id>" })
→ Returns full movie with metadata, scenes, and code blocks
```

Present a summary of existing scenes to the user.

### 3. Understand the Request

Ask the user what they want changed, or infer from context. Common operations:

- Rewrite narration for a scene
- Add new scenes at a specific position
- Remove scenes
- Reorder scenes
- Update code blocks (content, line ranges, highlights)
- Change view type
- Update metadata (title, description)

### 4. Make Surgical Updates

#### Update Movie Metadata
```
update_movie({ movieId: "<movie-id>", movie: { ...fullMovieWithUpdatedMetadata } })
```

#### Add a Scene
```
create_scene({ movieId: "<movie-id>", scene: { narration: "...", view: { ... } } })
```
After adding, use `reorder_scenes` to place it in the correct position.

#### Update a Scene (Partial)
```
patch_scene({ movieId: "<movie-id>", sceneId: "<scene-id>", scene: { narration: "Updated." } })
```
`patch_scene` only updates the fields you include.

#### Replace a Scene (Full)
```
update_scene({ movieId: "<movie-id>", sceneId: "<scene-id>", scene: { narration: "...", view: { ... }, timestamp: 0 } })
```

#### Delete a Scene
```
delete_scene({ movieId: "<movie-id>", sceneId: "<scene-id>" })
```

#### Reorder Scenes
```
reorder_scenes({ movieId: "<movie-id>", sceneIds: ["scene-3", "scene-1", "scene-2"] })
```

#### Manage Code Blocks
```
// Add
create_codeblock({ movieId: "...", sceneId: "...", block: { filePath: "src/file.ts", lineRanges: [...], changeType: "add", content: "..." } })

// Update
update_codeblock({ movieId: "...", sceneId: "...", blockId: "...", block: { filePath: "src/file.ts", lineRanges: [...], changeType: "modify", content: "...", parentId: null } })

// Delete
delete_codeblock({ movieId: "...", sceneId: "...", blockId: "..." })
```

### 5. Return Studio URL

Use the `studioUrl` returned by the API and prepend `$MERGE_MOVIES_URL` (default: `https://merge.mov`).

```
{MERGE_MOVIES_URL}{studioUrl}
```

## Common Update Patterns

- **Rewrite all narration:** `list_scenes`, iterate, `patch_scene` each with updated narration
- **Insert at position:** `create_scene` (appended), `list_scenes` to get IDs, splice new ID into desired position, `reorder_scenes`
- **Swap view type:** `update_scene` with a completely new view object
- **Add highlights:** `patch_scene` with full `view.animations` object including new highlights
- **Update code content:** Read the updated source file, `update_codeblock` with new content and lineRanges

## Scene Type Reference

### Code View
```json
{
  "type": "code",
  "layout": "single | side-by-side | stacked | inline-diff",
  "codeBlocks": [{
    "filePath": "src/file.ts",
    "lineRanges": [{ "start": 1, "end": 30 }],
    "changeType": "modify | add | delete | context",
    "content": "// actual file content"
  }],
  "animations": {
    "scroll": { "id": "s1", "linesPerSecond": 3, "pauses": [{ "lineNumber": 15, "durationMs": 2000 }] },
    "highlights": [{ "id": "h1", "lines": [5, 6, 7], "color": "rgba(255, 213, 79, 0.3)" }]
  }
}
```

### Slide View
```json
{
  "type": "slide",
  "backgroundColor": "#0d1117",
  "elements": [
    { "id": "t", "type": "text", "style": "title | body | bullet", "content": "...", "position": { "x": 10, "y": 30 } },
    { "id": "r", "type": "rect", "position": { "x": 5, "y": 35 }, "size": { "width": 20, "height": 15 }, "stroke": "#58a6ff", "text": "Label" },
    { "id": "c", "type": "circle", "position": { "x": 50, "y": 50 }, "size": { "width": 15 }, "fill": "#58a6ff" },
    { "id": "l", "type": "line", "position": { "x": 25, "y": 42 }, "endPosition": { "x": 35, "y": 42 }, "stroke": "#58a6ff" },
    { "id": "i", "type": "image", "src": "https://...", "position": { "x": 10, "y": 10 }, "size": { "width": 40 } }
  ]
}
```

### Terminal View
```json
{
  "type": "terminal",
  "title": "~/project",
  "theme": "generic | claude-code | codex",
  "inputAnimation": "type | fade | cut",
  "outputAnimation": "type | fade | cut",
  "entries": [{ "id": "e1", "command": "npm test", "output": "Tests passed", "exitCode": 0 }]
}
```

### React View
```json
{
  "type": "react",
  "backgroundColor": "#0d1117",
  "code": "const { useCurrentFrame, spring, interpolate, AbsoluteFill } = scope; ..."
}
```

Scope provides: `React`, `useCurrentFrame`, `useVideoConfig`, `spring`, `interpolate`, `Sequence`, `Series`, `AbsoluteFill`, `Img`

## Transitions

```json
{ "startTransition": { "type": "fade", "duration": 0.5 }, "endTransition": { "type": "fade", "duration": 0.3 } }
```

Types: `cut`, `fade`, `slide`, `zoom`
