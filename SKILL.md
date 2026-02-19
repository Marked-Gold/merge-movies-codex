---
name: merge-movies
description: Generate code walkthrough movies from git diffs using merge.mov
---

# Merge Movies

Generate code walkthrough movies from git diffs using the merge.mov REST API.

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

### 1. Get the Git Diff

```bash
git diff --name-status HEAD~3..HEAD   # scope first
git diff HEAD~3..HEAD                 # full diff
```

### 2. Analyze the Changes

Group changes into logical units by feature, file type, or layer. Plan a narrative: What problem is being solved? How does the solution work?

### 3. Plan Scenes

Create an outline before building. Each scene should focus on one concept. Use a mix of scene types (code, slide, terminal, react) for variety.

**Duration guidelines:** Simple changes: 3-5s. Complex logic: 8-12s. Architecture overviews: 10-15s.

### 4. Write Narration

Explain the "why" not just the "what." Use active voice. Connect to the bigger picture.

### 5. Create the Movie via API

```bash
# Create movie
MOVIE_RESPONSE=$(curl -s -X POST "https://merge.mov/api/movies" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '{
    "movie": {
      "metadata": { "title": "My Movie", "description": "What changed and why" },
      "scenes": []
    }
  }')
MOVIE_ID=$(echo "$MOVIE_RESPONSE" | jq -r '.id')

# Add scenes (repeat for each scene)
curl -s -X POST "https://merge.mov/api/movies/$MOVIE_ID/scenes" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '{ "narration": "...", "view": { ... } }'

echo "View at: https://studio.merge.mov/movie/$MOVIE_ID"
```

## Scene Types

### Code View

Display code with syntax highlighting. Always read the source file first to include proper context (3-5 lines before/after changes, enclosing function signatures).

```json
{
  "narration": "We update the validation logic.",
  "view": {
    "type": "code",
    "layout": "single",
    "codeBlocks": [{
      "filePath": "src/validators/user.ts",
      "lineRanges": [
        { "start": 38, "end": 40 },
        { "start": 44, "end": 55 }
      ],
      "changeType": "modify",
      "content": "// content covering all lines in lineRanges"
    }],
    "animations": {
      "highlights": [
        { "id": "h1", "lines": [50, 51, 52], "color": "rgba(255, 213, 79, 0.3)" }
      ]
    }
  }
}
```

**Layouts:** `single`, `side-by-side`, `stacked`, `inline-diff`

**Change types:** `modify` (existing file changed — most common), `add` (new file), `delete` (removed), `context` (unchanged, for reference)

**Animations:**
- **Scroll:** `"scroll": { "id": "s1", "linesPerSecond": 3, "pauses": [{ "lineNumber": 15, "durationMs": 2000 }] }`
- **Highlights:** `"highlights": [{ "id": "h1", "lines": [5, 6, 7], "color": "rgba(255, 213, 79, 0.3)" }]`

Highlight `lines` must use source file line numbers matching the code block's `lineRanges`.

### Slide View

Title cards, bullet lists, and diagrams with positioned elements on a 1920x1080 canvas. Positions are percentages (0-100).

```json
{
  "narration": "Let's walk through the changes.",
  "view": {
    "type": "slide",
    "elements": [
      { "id": "title", "type": "text", "style": "title", "content": "Authentication Overhaul", "position": { "x": 10, "y": 30 } },
      { "id": "body", "type": "text", "style": "bullet", "content": "JWT tokens\nRefresh flow\nRoute protection", "position": { "x": 10, "y": 50 }, "size": { "width": 80 } }
    ]
  }
}
```

**Text styles:** `title` (64px, bold, white), `body` (32px, #e6edf3), `bullet` (28px, blue dot prefix, splits on `\n`)

**Shape elements:** `rect`, `circle`, `line` — support `text`, `textColor`, `textFontSize`, `stroke`, `fill`, `borderRadius`

### Terminal View

Animated terminal sessions with command input and output.

```json
{
  "narration": "Install dependencies and run the tests.",
  "view": {
    "type": "terminal",
    "title": "~/project",
    "theme": "generic",
    "entries": [
      { "id": "e1", "command": "npm install", "output": "added 127 packages in 4s" },
      { "id": "e2", "command": "npm test", "output": "Tests: 12 passed", "exitCode": 0 }
    ],
    "inputAnimation": "type",
    "outputAnimation": "fade"
  }
}
```

**Themes:** `generic`, `claude-code`, `codex`

### React View

Custom animated scenes using React/JSX with Remotion APIs. The `code` field receives `scope` with: `useCurrentFrame`, `useVideoConfig`, `spring`, `interpolate`, `AbsoluteFill`, `Sequence`, `Series`, `Img`.

```json
{
  "narration": "Welcome to the walkthrough.",
  "view": {
    "type": "react",
    "code": "const { useCurrentFrame, spring, interpolate, AbsoluteFill } = scope;\nconst frame = useCurrentFrame();\nconst opacity = interpolate(frame, [0, 30], [0, 1], { extrapolateRight: 'clamp' });\nreturn <AbsoluteFill style={{ justifyContent: 'center', alignItems: 'center' }}><div style={{ opacity, fontSize: 80, color: '#58a6ff' }}>Hello</div></AbsoluteFill>;"
  }
}
```

## Transitions

```json
{
  "startTransition": { "type": "fade", "duration": 0.5 },
  "endTransition": { "type": "fade", "duration": 0.3 }
}
```

Types: `cut` (related scenes), `fade` (topic change), `slide` (sequential steps), `zoom` (drilling in)

## Tips

1. One PR = one movie — don't cover too much
2. Tell a story — beginning, middle, end
3. Mix scene types — code, slides, terminal, react
4. Scroll through long files instead of cramming
5. Use highlights to draw attention to key lines
6. Time scroll pauses to narration
7. Fade between topics, cut between related scenes
