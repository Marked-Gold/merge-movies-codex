---
name: create-movie
description: Create code walkthrough movies from git diffs, feature walkthroughs, architecture overviews, setup guides, or free-form narratives using merge.mov
---

# Create Movie

Create code walkthrough movies using the merge.mov REST API. Supports multiple creation paths beyond git diffs.

## Setup

Set the `MERGE_MOVIES_API_KEY` environment variable. Get a key from [merge.mov Settings](https://studio.merge.mov/settings).

All API calls use this pattern:

```bash
curl -s -X <METHOD> "https://merge.mov/api/<path>" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY" \
  -d '<json body>'
```

## Creation Paths

Determine the creation mode based on the user's request or conversation context:

| Signal | Mode | Source Material |
|--------|------|-----------------|
| Commit range (e.g., `HEAD~3..HEAD`) | Git diff | `git diff` output |
| `uncommitted` or working tree changes | Git diff | `git diff HEAD` |
| Branch name | Git diff | `git diff main..<branch>` |
| "walkthrough", "explain how X works" | Feature walkthrough | Read source files, trace execution flow |
| "architecture", "system overview" | Architecture overview | Explore codebase structure, explain the system |
| "setup", "getting started" | Setup guide | Read README, package.json, Dockerfiles, config |
| General description or free text | Free-form narrative | User describes the story, gather supporting files |

## Workflow

### 1. Gather Source Material

**Git diff modes:**
```bash
git diff --name-status HEAD~3..HEAD   # scope first
git diff HEAD~3..HEAD                 # full diff
```

**Feature walkthrough:** Use file search to find relevant source files — entry points, key modules, tests. Read each file to understand the implementation. Map out data/control flow through the feature.

**Architecture overview:** Explore directory structure, read entry points (package.json, main.ts, config files), identify layers (routes, services, models, utils), read representative files from each layer.

**Setup guide:** Read README, CONTRIBUTING, package.json, Dockerfile, docker-compose, config files. Identify prerequisites, install steps, environment variables, build commands.

**Free-form:** Ask the user what story they want to tell, gather supporting code and files.

### 2. Analyze and Plan Scenes

Group material into logical units. Plan a narrative arc:

**Git diffs:** Group by feature, file type, or layer. What problem is being solved? How does the solution work?

**Walkthroughs:** Follow execution flow — entry point, core logic, output. Build understanding progressively.

**Architecture:** Start high-level (system diagram), drill into layers, show how components connect.

**Setup:** Follow chronological setup order, show terminal commands alongside config files.

**Duration guidelines:** Simple changes: 3-5s. Complex logic: 8-12s. Architecture overviews: 10-15s.

### 3. Write Narration

Explain the "why" not just the "what." Use active voice. Connect to the bigger picture.

### 4. Read Source Files

Always read the actual source file before creating code view scenes. The diff tells you which lines changed; the file gives you the content with proper surrounding context. For non-diff modes, read files to get exact content for the lines you want to show.

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

For non-diff modes, omit `branch` and `commitRange` from metadata.

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

**Change types:** `modify` (existing file changed — most common), `add` (new file), `delete` (removed), `context` (unchanged, for reference — use for walkthrough/architecture scenes)

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

1. One topic per movie — don't cover too much
2. Tell a story — beginning, middle, end
3. Mix scene types — code, slides, terminal, react
4. Scroll through long files instead of cramming
5. Use highlights to draw attention to key lines
6. Time scroll pauses to narration
7. Fade between topics, cut between related scenes
8. For walkthroughs/architecture, use `context` changeType when showing existing code
