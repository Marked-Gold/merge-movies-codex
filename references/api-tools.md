# merge.mov REST API Reference

**Base URL:** `https://merge.mov`
**Auth:** `X-API-Key: $MERGE_MOVIES_API_KEY` header on all requests
**Content-Type:** `application/json`
**Studio:** `https://studio.merge.mov/movie/{movieId}` (viewing URL)

## Movies

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/movies` | List all movies |
| POST | `/api/movies` | Create movie |
| GET | `/api/movies/:id` | Get movie with all scenes |
| PUT | `/api/movies/:id` | Update full movie |
| DELETE | `/api/movies/:id` | Delete movie |

**Create movie body:**
```json
{
  "id": "optional-custom-id",
  "movie": {
    "metadata": {
      "title": "Required title",
      "description": "Required description",
      "repository": "optional/repo",
      "branch": "optional-branch",
      "commitRange": { "from": "abc123", "to": "def456" }
    },
    "scenes": []
  }
}
```

**Response:** `{ "id": "movie-id" }`

## Scenes

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/movies/:movieId/scenes` | List scenes |
| POST | `/api/movies/:movieId/scenes` | Create scene |
| GET | `/api/movies/:movieId/scenes/:sceneId` | Get scene |
| PATCH | `/api/movies/:movieId/scenes/:sceneId` | Partial update |
| PUT | `/api/movies/:movieId/scenes/:sceneId` | Full replace |
| DELETE | `/api/movies/:movieId/scenes/:sceneId` | Delete scene |
| POST | `/api/movies/:movieId/scenes/reorder` | Reorder scenes |

**Create scene body:**
```json
{
  "id": "optional-id",
  "title": "optional title",
  "narration": "Required narration text",
  "view": { "type": "code | slide | terminal | react | video", "..." : "..." },
  "timestamp": 0,
  "duration": 5,
  "startTransition": { "type": "fade", "duration": 0.5 },
  "endTransition": { "type": "fade", "duration": 0.3 }
}
```

**Reorder body:** `{ "sceneIds": ["scene-1", "scene-2", "scene-3"] }`

## Code Blocks

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/movies/:movieId/scenes/:sceneId/codeblocks` | List code blocks |
| POST | `/api/movies/:movieId/scenes/:sceneId/codeblocks` | Create code block |
| GET | `/api/movies/:movieId/scenes/:sceneId/codeblocks/:blockId` | Get code block |
| PUT | `/api/movies/:movieId/scenes/:sceneId/codeblocks/:blockId` | Update code block |
| DELETE | `/api/movies/:movieId/scenes/:sceneId/codeblocks/:blockId` | Delete code block |

**Create code block body:**
```json
{
  "id": "optional-id",
  "name": "optional label",
  "parentId": null,
  "filePath": "src/file.ts",
  "lineRanges": [{ "start": 1, "end": 30 }],
  "changeType": "modify",
  "content": "// actual file content for the line ranges",
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
