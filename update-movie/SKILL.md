---
name: update-movie
description: Modify an existing merge.mov movie — add, edit, reorder, or remove scenes
---

# Update Movie

Modify an existing merge.mov movie using the REST API. Add, edit, reorder, or remove scenes.

## Setup

Set the `MERGE_MOVIES_API_KEY` environment variable. Get a key from [merge.mov Settings](https://studio.merge.mov/settings).

All API calls use this pattern:

```bash
curl -s -X <METHOD> "https://merge.mov/api/<path>" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '<json body>'
```

## Workflow

### 1. Find the Target Movie

**By ID** (UUID-like string):
```bash
curl -s -X GET "https://merge.mov/api/movies/$MOVIE_ID" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY"
```

**By title search** — list movies and match:
```bash
curl -s -X GET "https://merge.mov/api/movies" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" | jq '.[] | { id, title: .metadata.title }'
```

**No input** — list movies and ask the user to pick.

### 2. Fetch Current State

Get the full movie with all scenes:

```bash
MOVIE=$(curl -s -X GET "https://merge.mov/api/movies/$MOVIE_ID" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY")
echo "$MOVIE" | jq '.scenes[] | { id, title, narration: .narration[:60], viewType: .view.type }'
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
```bash
curl -s -X PUT "https://merge.mov/api/movies/$MOVIE_ID" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '{ full movie object with updated metadata }'
```

#### Add a Scene
```bash
curl -s -X POST "https://merge.mov/api/movies/$MOVIE_ID/scenes" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '{ "narration": "...", "view": { ... } }'
```
After adding, use reorder to place it in the correct position.

#### Update a Scene (Partial)
```bash
curl -s -X PATCH "https://merge.mov/api/movies/$MOVIE_ID/scenes/$SCENE_ID" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '{ "narration": "Updated narration." }'
```
PATCH only updates the fields you include.

#### Replace a Scene (Full)
```bash
curl -s -X PUT "https://merge.mov/api/movies/$MOVIE_ID/scenes/$SCENE_ID" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '{ "narration": "...", "view": { ... } }'
```

#### Delete a Scene
```bash
curl -s -X DELETE "https://merge.mov/api/movies/$MOVIE_ID/scenes/$SCENE_ID" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY"
```

#### Reorder Scenes
```bash
curl -s -X POST "https://merge.mov/api/movies/$MOVIE_ID/scenes/reorder" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '{ "sceneIds": ["scene-3", "scene-1", "scene-2"] }'
```

#### Manage Code Blocks
```bash
# Add a code block
curl -s -X POST "https://merge.mov/api/movies/$MOVIE_ID/scenes/$SCENE_ID/codeblocks" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '{ "filePath": "src/file.ts", "lineRanges": [{"start": 1, "end": 20}], "changeType": "add", "content": "..." }'

# Update a code block
curl -s -X PUT "https://merge.mov/api/movies/$MOVIE_ID/scenes/$SCENE_ID/codeblocks/$BLOCK_ID" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '{ "filePath": "src/file.ts", "lineRanges": [{"start": 10, "end": 30}], "changeType": "modify", "content": "..." }'

# Delete a code block
curl -s -X DELETE "https://merge.mov/api/movies/$MOVIE_ID/scenes/$SCENE_ID/codeblocks/$BLOCK_ID" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY"
```

### 5. Return Studio URL

```
https://studio.merge.mov/movie/{MOVIE_ID}
```

## Common Update Patterns

- **Rewrite all narration:** Fetch scenes, iterate, PATCH each with updated narration
- **Insert at position:** POST new scene, GET all scene IDs, splice new ID into desired position, POST reorder
- **Swap view type:** PUT the scene with a completely new view object
- **Add highlights:** PATCH the scene with the full `view.animations` object including new highlights
- **Update code content:** Read the updated source file, PUT the code block with new content and lineRanges

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
