---
name: create-movie
description: Create code walkthrough movies from git diffs, feature walkthroughs, architecture overviews, setup guides, or free-form narratives using merge.mov
---

# Create Movie

Create code walkthrough movies using the merge.mov MCP tools. Supports multiple creation paths beyond git diffs.

## Setup

The `merge-movies` MCP server must be connected. Authentication is handled via the `MERGE_MOVIES_API_KEY` environment variable passed through the MCP transport.

If the key is missing, tell the user to create one at [merge.mov Settings](https://merge.mov/settings).

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

### 5. Create the Movie via MCP Tools

```
// Create movie
create_movie({
  movie: {
    metadata: { title: "My Movie", description: "What changed and why" },
    scenes: []
  }
})
→ Returns { id, studioUrl }

// Add scenes (repeat for each scene)
create_scene({
  movieId: "<movie-id>",
  scene: { narration: "...", view: { ... } }
})

// Return studio URL — use $MERGE_MOVIES_URL if set, otherwise https://merge.mov
// {MERGE_MOVIES_URL}{studioUrl}
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

**Use slide views only for simple title cards and closing slides.** For bullet lists, diagrams, architecture flows, or anything that benefits from animation, use React views instead.

```json
{
  "narration": "Let's walk through the changes.",
  "view": {
    "type": "slide",
    "elements": [
      { "id": "title", "type": "text", "style": "title", "content": "Authentication Overhaul", "position": { "x": 10, "y": 30 } },
      { "id": "subtitle", "type": "text", "style": "body", "content": "Adding JWT tokens, refresh flow, and route protection", "position": { "x": 10, "y": 50 } }
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

**React views are the preferred scene type for all visual/explanatory content** — bullet lists, diagrams, architecture flows, feature overviews, and anything that isn't pure code or terminal output. The `code` field receives `scope` with: `useCurrentFrame`, `useVideoConfig`, `spring`, `interpolate`, `AbsoluteFill`, `Sequence`, `Series`, `Img`.

**Example: Animated bullet list**

```json
{
  "narration": "Here's what we'll cover.",
  "view": {
    "type": "react",
    "code": "const { useCurrentFrame, interpolate, AbsoluteFill } = scope;\nconst frame = useCurrentFrame();\nconst items = ['JWT tokens', 'Refresh flow', 'Route protection'];\nconst titleOpacity = interpolate(frame, [0, 20], [0, 1], { extrapolateRight: 'clamp' });\nreturn (\n  <AbsoluteFill style={{ padding: '96px 120px', backgroundColor: '#0d1117' }}>\n    <div style={{ opacity: titleOpacity, fontSize: 64, fontWeight: 700, color: '#fff', marginBottom: 48 }}>What Changed</div>\n    {items.map((item, i) => {\n      const delay = 20 + i * 15;\n      const opacity = interpolate(frame, [delay, delay + 15], [0, 1], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });\n      const tx = interpolate(frame, [delay, delay + 15], [30, 0], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });\n      return <div key={i} style={{ opacity, transform: `translateX(${tx}px)`, fontSize: 28, color: '#e6edf3', marginBottom: 20, display: 'flex', gap: 16 }}><span style={{ color: '#58a6ff' }}>&#x2022;</span>{item}</div>;\n    })}\n  </AbsoluteFill>\n);"
  }
}
```

**Example: Simple animated intro**

```json
{
  "narration": "Welcome to the walkthrough.",
  "view": {
    "type": "react",
    "code": "const { useCurrentFrame, spring, interpolate, AbsoluteFill, useVideoConfig } = scope;\nconst frame = useCurrentFrame();\nconst { fps } = useVideoConfig();\nconst opacity = interpolate(frame, [0, 30], [0, 1], { extrapolateRight: 'clamp' });\nconst scale = spring({ frame, fps, config: { damping: 200 } });\nreturn <AbsoluteFill style={{ justifyContent: 'center', alignItems: 'center' }}><div style={{ opacity, transform: `scale(${scale})`, fontSize: 80, color: '#58a6ff' }}>Hello</div></AbsoluteFill>;"
  }
}
```

**Tips:** All animation must be frame-driven (`useCurrentFrame`). Use inline styles only. Canvas is 1920x1080 at 30fps. Prefer React views over slide views for anything beyond simple title cards.

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
3. Prefer React views for visual content (bullet lists, diagrams, overviews) — only use slides for title cards and closing slides
4. Scroll through long files instead of cramming
5. Use highlights to draw attention to key lines
6. Time scroll pauses to narration
7. Fade between topics, cut between related scenes
8. For walkthroughs/architecture, use `context` changeType when showing existing code
