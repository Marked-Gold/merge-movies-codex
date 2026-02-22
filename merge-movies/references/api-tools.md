# merge.mov MCP Tools Reference

**MCP Server:** `merge-movies`
**Transport:** Streamable HTTP
**URL:** `https://merge.mov/api/mcp`
**Auth:** OAuth 2.1 — authentication handled automatically by MCP transport (browser login on first use)
**Studio:** `{MERGE_MOVIES_URL}/studio/{movieId}` (viewing URL, use `$MERGE_MOVIES_URL` or default `https://merge.mov`)

## Movies

| Tool | Input | Description |
|------|-------|-------------|
| `list_movies` | `{}` | List all movies — returns `[{ id, title, updatedAt }]` |
| `get_movie` | `{ movieId }` | Get movie with all scenes and code blocks |
| `create_movie` | `{ movie }` | Create movie — returns `{ id, studioUrl }` |
| `update_movie` | `{ movieId, movie }` | Full replacement of a movie |
| `delete_movie` | `{ movieId }` | Delete a movie |

**Movie object:**
```json
{
  "metadata": {
    "title": "Required title",
    "description": "Required description",
    "repository": "optional/repo",
    "branch": "optional-branch",
    "commitRange": { "from": "abc123", "to": "def456" }
  },
  "scenes": []
}
```

## Scenes

| Tool | Input | Description |
|------|-------|-------------|
| `list_scenes` | `{ movieId }` | List all scenes in a movie |
| `get_scene` | `{ movieId, sceneId }` | Get a single scene |
| `create_scene` | `{ movieId, scene }` | Create a new scene |
| `update_scene` | `{ movieId, sceneId, scene }` | Full replacement of a scene |
| `patch_scene` | `{ movieId, sceneId, scene }` | Partial update (only provided fields) |
| `delete_scene` | `{ movieId, sceneId }` | Delete a scene |
| `reorder_scenes` | `{ movieId, sceneIds }` | Reorder scenes by ID array |

**Scene object (for create/update):**
```json
{
  "narration": "Required narration text",
  "view": { "type": "code | slide | terminal | react | video", "..." : "..." },
  "title": "optional title",
  "timestamp": 0,
  "duration": 5,
  "startTransition": { "type": "fade", "duration": 0.5 },
  "endTransition": { "type": "fade", "duration": 0.3 }
}
```

## Code Blocks

| Tool | Input | Description |
|------|-------|-------------|
| `list_codeblocks` | `{ movieId, sceneId }` | List code blocks in a scene |
| `get_codeblock` | `{ movieId, sceneId, blockId }` | Get a single code block |
| `create_codeblock` | `{ movieId, sceneId, block }` | Create a code block |
| `update_codeblock` | `{ movieId, sceneId, blockId, block }` | Update a code block |
| `delete_codeblock` | `{ movieId, sceneId, blockId }` | Delete a code block |

**Code block object:**
```json
{
  "filePath": "src/file.ts",
  "lineRanges": [{ "start": 1, "end": 30 }],
  "changeType": "modify",
  "content": "// actual file content for the line ranges",
  "parentId": null,
  "lineOrder": [1, 2, 3]
}
```

**Required fields:** `filePath`, `lineRanges`, `changeType`

## View Types

### Code View
```json
{
  "type": "code",
  "layout": "single | side-by-side | stacked | inline-diff",
  "codeBlocks": [{ "filePath": "...", "lineRanges": [...], "changeType": "...", "content": "..." }],
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
  "entries": [
    { "id": "e1", "command": "npm test", "output": "Tests passed", "exitCode": 0, "entryType": "generic" }
  ]
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
