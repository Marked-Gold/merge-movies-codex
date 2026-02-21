# Merge Movies — Codex Skill

Create and update code walkthrough movies using [merge.mov](https://merge.mov).

## Install

```bash
$skill-installer install https://github.com/Marked-Gold/merge-movies-codex/tree/main/merge-movies
```

Restart Codex after installation so the new skill is loaded.

## Setup

1. Sign up at [merge.mov](https://merge.mov)
2. Go to [Settings](https://merge.mov/settings) and create an API key
3. Set the environment variable:

```bash
export MERGE_MOVIES_API_KEY="mm_your_key_here"
```

4. Add MCP config — this repo includes `.mcp.json` in `merge-movies/.mcp.json`; copy it to your project if needed
5. Restart Codex, then run `/mcp` and verify `merge-movies` is listed

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
