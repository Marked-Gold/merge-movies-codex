---
name: merge-movies
description: Create and update code walkthrough movies using merge.mov. Use when the user wants to make a new movie from code changes or modify an existing one.
metadata:
  short-description: Create & edit merge.mov walkthrough movies
---

# merge-movies

## Overview

This skill creates and updates code walkthrough movies using [merge.mov](https://merge.mov). It provides two tools — **Create Movie** for building new movies from git diffs, feature walkthroughs, architecture overviews, setup guides, or free-form narratives, and **Update Movie** for modifying existing movies (adding, editing, reordering, or removing scenes).

Movies are built from scenes — each scene has narration and a visual view (code, terminal, slide, or React animation). The merge.mov MCP server provides all the API tools needed to manage movies, scenes, and code blocks.

## Prerequisites

- merge-movies MCP server must be connected via streamable HTTP
- Authentication is handled automatically via OAuth (browser login on first use)

## Required Workflow

**Follow these steps in order. Do not skip steps.**

### Step 0: Set up merge-movies MCP (if not already configured)

If `/mcp` does not list `merge-movies`, or any MCP call fails because the server is not connected, pause and set it up:

1. Add the MCP server:
   ```
   codex mcp add merge-movies -- npx mcp-remote https://merge.mov/api/mcp
   ```

2. Restart Codex and run `/mcp` — a browser window will open for you to log in and authorize access.

Authentication is handled automatically via OAuth. No API keys or environment variables needed.

Use `$MERGE_MOVIES_URL` if set, otherwise default to `https://merge.mov`.

### Step 1: Clarify Intent

Determine whether the user wants to **create** a new movie or **update** an existing one. Signals:

| Signal | Tool |
|--------|------|
| "make a movie", "create a walkthrough", commit range, branch name, "uncommitted" | Create Movie |
| "edit my movie", "update narration", "add a scene", movie ID or title reference | Update Movie |
| Ambiguous | Ask the user |

### Step 2: Execute the Tool

Follow the appropriate tool workflow below — **Create Movie** or **Update Movie**.

### Step 3: Return the Studio URL

After creating or updating, return the studio URL so the user can review:

```
{MERGE_MOVIES_URL}{studioUrl}
```

Use `$MERGE_MOVIES_URL` if set, otherwise `https://merge.mov`.

---

## Tools

### Create Movie

Create code walkthrough movies from various source materials.

#### Determine Creation Mode

Parse the user's request to detect the mode:

| Signal | Mode | Source Material |
|--------|------|-----------------|
| Commit range (e.g., `HEAD~3..HEAD`) | Git diff | `git diff` output |
| `uncommitted` or working tree changes | Git diff | `git diff HEAD` |
| Branch name | Git diff | `git diff main..<branch>` |
| "walkthrough", "explain how X works" | Feature walkthrough | Read source files, trace execution flow |
| "architecture", "system overview" | Architecture overview | Explore codebase structure |
| "setup", "getting started" | Setup guide | Read README, package.json, config files |
| General description or free text | Free-form narrative | User describes the story |

If the mode is ambiguous, ask the user to clarify.

#### Gather Source Material

**Git diff modes:**
```bash
git diff --name-status HEAD~3..HEAD   # scope first
git diff HEAD~3..HEAD                 # full diff
```

**Feature walkthrough:** Use file search to find relevant source files — entry points, key modules, tests. Read each file to understand the implementation. Map out data/control flow through the feature.

**Architecture overview:** Explore directory structure, read entry points (package.json, main.ts, config files), identify layers (routes, services, models, utils), read representative files from each layer.

**Setup guide:** Read README, CONTRIBUTING, package.json, Dockerfile, docker-compose, config files. Identify prerequisites, install steps, environment variables, build commands.

**Free-form:** Ask the user what story they want to tell, gather supporting code and files.

#### Analyze and Plan Scenes

Create a scene outline before building. Group material into logical scenes:

- **Git diffs:** Group by feature, file type, or layer. What problem? What solution? What result?
- **Walkthroughs:** Follow execution flow — entry point, core logic, output. Build understanding progressively.
- **Architecture:** Start high-level (system diagram), drill into layers, show how components connect.
- **Setup:** Follow chronological setup order, show terminal commands alongside config files.

**Duration guidelines:** Simple changes: 3-5s. Complex logic: 8-12s. Architecture overviews: 10-15s.

#### Write Narration

- Explain the "why" not just the "what"
- Use active voice ("We add..." not "A function is added...")
- Connect changes to the bigger picture

**Bad:** "Here we see the addition of a new function called handleSubmit."
**Good:** "The handleSubmit function validates user input before sending it to the API, preventing invalid data from reaching the server."

#### Read Source Files

**Always read the actual source file** before creating code view scenes. The diff tells you which lines changed; the file gives you the content with proper surrounding context. For non-diff modes, read files to get exact content for the lines you want to show.

#### Create the Movie

1. Create the movie:

```
create_movie({
  movie: {
    metadata: {
      title: "Add User Authentication",
      description: "JWT-based auth with route protection",
      repository: "acme/web-app",
      branch: "feature/auth",
      commitRange: { from: "abc123", to: "def456" }
    },
    scenes: []
  }
})
→ Returns { id, studioUrl }
```

For non-diff modes, omit `branch` and `commitRange`.

2. Add scenes using `create_scene` (see Scene Types below). Always include a `title` — a short label (2-5 words) for the scene:

```
create_scene({
  movieId: "<movie-id>",
  scene: {
    title: "Email Validation",
    narration: "We update the validation logic to check email format.",
    view: { type: "code", layout: "single", codeBlocks: [...] }
  }
})
```

Use a mix of scene types: code views for implementation, terminal views for CLI demos, **React views for most visual/explanatory scenes** (bullet lists, diagrams, overviews). Reserve slide views only for simple title cards and closing slides.

---

### Update Movie

Modify an existing movie — add, edit, reorder, or remove scenes.

#### Find the Target Movie

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

#### Fetch Current State

```
get_movie({ movieId: "<movie-id>" })
→ Returns full movie with metadata, scenes, and code blocks
```

Present a summary of existing scenes to the user: scene order, types, and narration snippets.

#### Understand the Request

Ask the user what they want changed, or infer from context. Common operations:
- Rewrite narration for a scene
- Add new scenes at a specific position
- Remove scenes
- Reorder scenes
- Update code blocks (content, line ranges, highlights)
- Change view type
- Update metadata (title, description)

#### Make Surgical Updates

**Update metadata:**
```
update_movie({ movieId: "<movie-id>", movie: { ...fullMovieWithUpdatedMetadata } })
```

**Add a scene:**
```
create_scene({ movieId: "<movie-id>", scene: { title: "New Scene", narration: "...", view: { ... } } })
```
After adding, use `reorder_scenes` to place it in the correct position.

**Partial update (narration, etc.):**
```
patch_scene({ movieId: "<movie-id>", sceneId: "<scene-id>", scene: { narration: "Updated." } })
```

**Full replacement:**
```
update_scene({ movieId: "<movie-id>", sceneId: "<scene-id>", scene: { narration: "...", view: { ... }, timestamp: 0 } })
```

**Delete a scene:**
```
delete_scene({ movieId: "<movie-id>", sceneId: "<scene-id>" })
```

**Reorder scenes:**
```
reorder_scenes({ movieId: "<movie-id>", sceneIds: ["scene-3", "scene-1", "scene-2"] })
```

**Manage code blocks:**
```
// Add
create_codeblock({ movieId: "...", sceneId: "...", block: { filePath: "src/file.ts", lineRanges: [...], changeType: "add", content: "..." } })

// Update
update_codeblock({ movieId: "...", sceneId: "...", blockId: "...", block: { filePath: "src/file.ts", lineRanges: [...], changeType: "modify", content: "..." } })

// Delete
delete_codeblock({ movieId: "...", sceneId: "...", blockId: "..." })
```

#### Common Update Patterns

- **Rewrite all narration:** `list_scenes`, iterate, `patch_scene` each with updated narration
- **Insert at position:** `create_scene` (appended), `list_scenes` to get IDs, splice new ID into desired position, `reorder_scenes`
- **Swap view type:** `update_scene` with a completely new view object
- **Add highlights:** `patch_scene` with full `view.animations` object including new highlights
- **Update code content:** Read the updated source file, `update_codeblock` with new content and lineRanges

---

## MCP Tools

All tools are available automatically via the `merge-movies` MCP server. Authentication is handled automatically via OAuth 2.1 (browser login on first use).

### Movies

| Tool | Input | Description |
|------|-------|-------------|
| `list_movies` | `{}` | List all movies — returns `[{ id, title, updatedAt }]` |
| `get_movie` | `{ movieId }` | Get movie with all scenes and code blocks |
| `create_movie` | `{ movie }` | Create movie — returns `{ id, studioUrl }` |
| `update_movie` | `{ movieId, movie }` | Full replacement of a movie |
| `delete_movie` | `{ movieId }` | Delete a movie |

### Scenes

| Tool | Input | Description |
|------|-------|-------------|
| `list_scenes` | `{ movieId }` | List all scenes in a movie |
| `get_scene` | `{ movieId, sceneId }` | Get a single scene |
| `create_scene` | `{ movieId, scene }` | Create a new scene |
| `update_scene` | `{ movieId, sceneId, scene }` | Full replacement of a scene |
| `patch_scene` | `{ movieId, sceneId, scene }` | Partial update (only provided fields) |
| `delete_scene` | `{ movieId, sceneId }` | Delete a scene |
| `reorder_scenes` | `{ movieId, sceneIds }` | Reorder scenes by ID array |

### Code Blocks

| Tool | Input | Description |
|------|-------|-------------|
| `list_codeblocks` | `{ movieId, sceneId }` | List code blocks in a scene |
| `get_codeblock` | `{ movieId, sceneId, blockId }` | Get a single code block |
| `create_codeblock` | `{ movieId, sceneId, block }` | Create a code block |
| `update_codeblock` | `{ movieId, sceneId, blockId, block }` | Update a code block |
| `delete_codeblock` | `{ movieId, sceneId, blockId }` | Delete a code block |

### Videos

| Tool | Input | Description |
|------|-------|-------------|
| `create_video_upload` | `{ movieId, filename?, mimeType? }` | Get a presigned URL to upload a video file. Returns `{ presignedUploadUrl, videoSource }` |

---

## Scene Types

### Code View

Display code with syntax highlighting. Always read the source file first to include proper context (3-5 lines before/after changes, enclosing function signatures).

```json
{
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
```

**Layouts:** `single`, `side-by-side`, `stacked`, `inline-diff`

**Change types:**
- `modify` — existing file changed (most common, even when showing added lines within an existing file)
- `add` — brand-new file
- `delete` — file or code removed
- `context` — unchanged code shown for reference (use for walkthrough/architecture scenes)

### Slide View

**Use slide views only for simple title cards and closing slides.** For bullet lists, diagrams, or anything with animation, use React views instead.

```json
{
  "type": "slide",
  "elements": [
    { "id": "title", "type": "text", "style": "title", "content": "Authentication Overhaul", "position": { "x": 10, "y": 30 } },
    { "id": "subtitle", "type": "text", "style": "body", "content": "Adding JWT tokens and route protection", "position": { "x": 10, "y": 50 } }
  ]
}
```

**Text styles:** `title` (64px, bold, white), `body` (32px, #e6edf3), `bullet` (28px, blue dot prefix, splits on `\n`)

**Shape elements:** `rect`, `circle`, `line` — support `text`, `textColor`, `textFontSize`, `stroke`, `fill`, `borderRadius`

**Image elements:** `type: "image"` with `src` URL, `position`, and `size`

### Terminal View

Animated terminal sessions with command input and output.

```json
{
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
```

**Themes:** `generic`, `claude-code`, `codex`
**Input animations:** `type` (typewriter), `fade`, `cut` (instant)
**Output animations:** `type`, `fade`, `cut`

### React View

**Preferred scene type for all visual/explanatory content** — bullet lists, diagrams, architecture flows, feature overviews. The `code` field is the body of a React function component that receives `scope` with: `React`, `useCurrentFrame`, `useVideoConfig`, `spring`, `interpolate`, `AbsoluteFill`, `Sequence`, `Series`, `Img`.

**Animated bullet list:**

```json
{
  "narration": "Here's what we'll cover.",
  "view": {
    "type": "react",
    "code": "const { useCurrentFrame, interpolate, AbsoluteFill } = scope;\nconst frame = useCurrentFrame();\nconst items = ['JWT tokens', 'Refresh flow', 'Route protection'];\nconst titleOpacity = interpolate(frame, [0, 20], [0, 1], { extrapolateRight: 'clamp' });\nreturn (\n  <AbsoluteFill style={{ padding: '96px 120px', backgroundColor: '#0d1117' }}>\n    <div style={{ opacity: titleOpacity, fontSize: 64, fontWeight: 700, color: '#fff', marginBottom: 48 }}>What Changed</div>\n    {items.map((item, i) => {\n      const delay = 20 + i * 15;\n      const opacity = interpolate(frame, [delay, delay + 15], [0, 1], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });\n      const tx = interpolate(frame, [delay, delay + 15], [30, 0], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });\n      return <div key={i} style={{ opacity, transform: `translateX(${tx}px)`, fontSize: 28, color: '#e6edf3', marginBottom: 20, display: 'flex', gap: 16 }}><span style={{ color: '#58a6ff' }}>•</span>{item}</div>;\n    })}\n  </AbsoluteFill>\n);"
  }
}
```

**Architecture diagram:**

```json
{
  "narration": "The request flows from client through middleware to the handler.",
  "view": {
    "type": "react",
    "code": "const { useCurrentFrame, interpolate, spring, useVideoConfig, AbsoluteFill } = scope;\nconst frame = useCurrentFrame();\nconst { fps } = useVideoConfig();\nconst boxes = [\n  { label: 'Client', color: '#58a6ff', x: 160 },\n  { label: 'Auth Middleware', color: '#d2a8ff', x: 600 },\n  { label: 'API Handler', color: '#3fb950', x: 1100 }\n];\nreturn (\n  <AbsoluteFill style={{ justifyContent: 'center', alignItems: 'center', backgroundColor: '#0d1117' }}>\n    <div style={{ fontSize: 48, fontWeight: 700, color: '#fff', position: 'absolute', top: 96 }}>Request Flow</div>\n    {boxes.map((box, i) => {\n      const delay = i * 20;\n      const scale = spring({ frame: frame - delay, fps, config: { damping: 200 } });\n      const opacity = interpolate(frame, [delay, delay + 10], [0, 1], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });\n      return <div key={i} style={{ position: 'absolute', left: box.x, top: 420, opacity, transform: `scale(${scale})`, border: `2px solid ${box.color}`, borderRadius: 12, padding: '24px 36px', color: box.color, fontSize: 24, fontWeight: 600 }}>{box.label}</div>;\n    })}\n    {[0, 1].map(i => {\n      const arrowDelay = 10 + i * 20;\n      const opacity = interpolate(frame, [arrowDelay + 10, arrowDelay + 20], [0, 1], { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' });\n      const x = i === 0 ? 380 : 870;\n      return <div key={`a${i}`} style={{ position: 'absolute', left: x, top: 446, opacity, color: '#58a6ff', fontSize: 32 }}>→</div>;\n    })}\n  </AbsoluteFill>\n);"
  }
}
```

**React view tips:**
- All animation must be frame-driven (`useCurrentFrame`), not time-based or stateful
- Use inline styles only — no CSS imports or className
- Canvas is 1920x1080 at 30fps. A 5-second scene = 150 frames
- Use `AbsoluteFill` as root container
- Use literal Unicode characters, not escape sequences — `\u2713` renders as literal text, not a checkmark. Paste actual characters: `✓`, `→`, `•`

---

## Code Context Guidelines

When creating code view scenes, provide proper context so viewers understand where changes occur.

### Rules

1. **Always read the source file first** — never paste raw diff hunks. The diff tells you *which* lines changed; the file gives you the *content* with surrounding context.
2. **Include 3-5 context lines before and after** each change
3. **Show the enclosing structure** — include function/class signatures. Use separate lineRanges for distant headers (creates `...` gap separator).
4. **Content must cover ALL lines in lineRanges** — the line count must match exactly
5. **Use actual source line numbers** in `lineRanges` and `highlights`

### Example

**Bad** (no context):
```json
{
  "lineRanges": [{ "start": 47, "end": 50 }],
  "content": "  const email = input.email.trim();\n  if (!isValidEmail(email)) {\n    throw new ValidationError('Invalid email format');\n  }"
}
```

**Good** (enclosing function + context):
```json
{
  "lineRanges": [
    { "start": 38, "end": 40 },
    { "start": 44, "end": 55 }
  ],
  "changeType": "modify",
  "content": "export function validateUser(input: UserInput): ValidationResult {\n  const errors: string[] = [];\n\n  const name = input.name?.trim();\n  if (!name || name.length < 2) {\n    errors.push('Name must be at least 2 characters');\n  }\n  const email = input.email.trim();\n  if (!isValidEmail(email)) {\n    throw new ValidationError('Invalid email format');\n  }\n  const age = Number(input.age);\n  if (isNaN(age) || age < 0 || age > 150) {\n    errors.push('Invalid age');\n  }\n  return { valid: errors.length === 0, errors };",
  "animations": {
    "highlights": [
      { "id": "h1", "lines": [50, 51, 52, 53], "color": "rgba(255, 213, 79, 0.3)" }
    ]
  }
}
```

---

## Animations

### Scrolling

For files too long to fit on screen:

```json
{
  "animations": {
    "scroll": {
      "id": "scroll1",
      "linesPerSecond": 3,
      "pauses": [
        { "lineNumber": 15, "durationMs": 2000 },
        { "lineNumber": 42, "durationMs": 3000 }
      ]
    }
  }
}
```

- `linesPerSecond`: 0.1-50, recommend 2-5 for readability
- A 2-second pause at the start is automatically added
- If you omit scroll animation, long content auto-scrolls at 1.5 lines/sec

### Highlights

```json
{
  "animations": {
    "highlights": [
      { "id": "h1", "lines": [5, 6, 7], "color": "rgba(255, 213, 79, 0.3)" }
    ]
  }
}
```

**Highlight fields:** `id` (required), `lines` (required, source line numbers), `color` (RGBA), `targetBlockId`, `startTimeMs`, `endTimeMs`, `glow`, `size`, `focus`

**Preset colors:** Yellow `rgba(255, 213, 79, 0.3)`, Blue `rgba(96, 165, 250, 0.3)`, Purple `rgba(192, 132, 252, 0.3)`, Orange `rgba(251, 146, 60, 0.3)`, Cyan `rgba(34, 211, 238, 0.3)`, Pink `rgba(244, 114, 182, 0.3)`

**Timed highlights:**
```json
{ "id": "h1", "lines": [3, 4, 5], "color": "rgba(96, 165, 250, 0.3)", "startTimeMs": 0, "endTimeMs": 3000 },
{ "id": "h2", "lines": [12, 13, 14], "color": "rgba(244, 114, 182, 0.3)", "startTimeMs": 3000, "endTimeMs": 6000 }
```

---

## Transitions

```json
{
  "startTransition": { "type": "fade", "duration": 0.5 },
  "endTransition": { "type": "fade", "duration": 0.3 }
}
```

| Type | When to use |
|------|-------------|
| `cut` | Between closely related scenes (same topic) |
| `fade` | Between distinct topics or for dramatic effect |
| `slide` | For sequential steps or progression |
| `zoom` | For drilling into details |

---

## Recording Browser Demos

You can record browser demos using Playwright and include them as VideoView scenes. This is useful for showing UI workflows, bug fixes, or feature demos.

Scripts and videos are persisted in `.merge-movies/demos/` so they can be reviewed, re-run, or debugged.

### Setup

```bash
npm install playwright 2>/dev/null && npx playwright install chromium --with-deps 2>/dev/null || npx playwright install chromium
```

### Write the Playwright Script

Choose a descriptive kebab-case name for the demo (e.g., `login-flow`, `search-feature`). Write the script to `.merge-movies/demos/<name>.mjs`:

```javascript
// .merge-movies/demos/<name>.mjs
import { chromium } from 'playwright';
import { dirname } from 'path';
import { fileURLToPath } from 'url';

const __dirname = dirname(fileURLToPath(import.meta.url));

const browser = await chromium.launch();
const context = await browser.newContext({
  recordVideo: { dir: __dirname, size: { width: 1280, height: 720 } },
  viewport: { width: 1280, height: 720 },
});
const page = await context.newPage();

// Customize: navigate, click, fill forms, etc.
await page.goto('http://localhost:3000');
await page.waitForTimeout(2000);

await context.close();
const videoPath = await page.video().path();
console.log('VIDEO_PATH:' + videoPath);
await browser.close();
```

### Run the Script

```bash
node .merge-movies/demos/<name>.mjs
```

Parse the video file path from the `VIDEO_PATH:` line in stdout. The `.webm` file will be saved in `.merge-movies/demos/` alongside the script.

If the script fails, fix the file and re-run — no need to rewrite from scratch.

### Upload and Create Scene

1. Get a presigned upload URL:
```
create_video_upload({ movieId: "<movie-id>" })
```

2. Upload the video:
```bash
curl -X PUT "<presignedUploadUrl>" -H "Content-Type: video/webm" --data-binary @<video-path>
```

3. Create the VideoView scene:
```
create_scene({
  movieId: "<movie-id>",
  scene: {
    title: "Demo: <feature>",
    narration: "<what the video shows>",
    view: { type: "video", source: "<videoSource>", aspectRatio: 1.778 }
  }
})
```

The script and video remain in `.merge-movies/demos/` for future reference. This folder is gitignored.

**Tips:**
- Keep demos short (10-30 seconds)
- Use `page.waitForTimeout(ms)` between actions for pacing
- Use `page.waitForSelector(selector)` instead of fixed waits when possible
- **Avoid repeated logins** — use Playwright's `storageState` to save the browser session after the first login and reuse it:
  ```javascript
  await context.storageState({ path: '.merge-movies/demos/auth-state.json' });
  // Later: browser.newContext({ storageState: '.merge-movies/demos/auth-state.json', ... })
  ```

---

## Tips for Great Movies

1. **Keep it focused** — one topic per movie, don't cover too much
2. **Tell a story** — beginning (context/problem), middle (solution), end (result/demo)
3. **Show, don't just tell** — let the code speak when possible
4. **Prefer React views for visual content** — bullet lists, diagrams, overviews, architecture flows. Only use slides for title cards and closing slides.
5. **Mix scene types** — alternate between code, react, and terminal views to keep viewers engaged
6. **Use animations for long files** — scroll through large files instead of cramming
7. **Time animations to narration** — sync scroll pauses with what you're explaining
8. **Use transitions** — fade between unrelated scenes, cut between related ones
9. **Use highlights** — draw attention to key lines with colored backgrounds
10. **Scene ordering** — start with context, introduce the problem, show the solution, demonstrate the result
