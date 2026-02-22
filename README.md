# Merge Movies — Codex Skill

Create and update code walkthrough movies using [merge.mov](https://merge.mov).

## Install

```bash
$skill-installer install https://github.com/Marked-Gold/merge-movies-codex/tree/main/merge-movies
```

Restart Codex after installation so the new skill is loaded.

## Setup

1. Sign up at [merge.mov](https://merge.mov)
2. Connect the MCP server:

```bash
codex mcp add merge-movies -- npx mcp-remote https://merge.mov/api/mcp
```

3. Restart Codex, then run `/mcp` — a browser window will open for you to log in
4. Authentication is handled automatically via OAuth — no API keys or environment variables needed

## Skill

### `merge-movies` — Create & update movies

A single skill with two tools:

**Create Movie** — build new movies from:
- **Git diffs** — from commit ranges, uncommitted changes, or branch comparisons
- **Feature walkthroughs** — explain how a feature works by reading source files
- **Architecture overviews** — explore and explain the system structure
- **Setup guides** — document how to set up and run the project
- **Free-form narratives** — tell any story with code, slides, and animations

**Update Movie** — modify existing movies:
- Add, edit, reorder, or remove scenes
- Update narration, code blocks, or view types
- Change metadata (title, description)

## What It Does

The skill teaches Codex to create and modify code walkthrough videos by calling merge.mov via MCP tools. Movies can include code views with syntax highlighting, slide views for title cards, terminal demos, and custom React animations.

View your movies at [merge.mov](https://merge.mov).

## License

MIT
