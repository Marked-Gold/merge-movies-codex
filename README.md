# Merge Movies — Codex Skill

Create and update code walkthrough movies using [merge.mov](https://merge.mov).

## Install

```bash
$skill-installer install https://github.com/Marked-Gold/merge-movies-codex
```

## Setup

1. Sign up at [merge.mov](https://merge.mov)
2. Go to [Settings](https://merge.mov/settings) and create an API key
3. Set the environment variable:

```bash
export MERGE_MOVIES_API_KEY="mm_your_key_here"
```

## Skills

### `create-movie` — Create a new movie

Supports multiple creation paths:
- **Git diffs** — from commit ranges, uncommitted changes, or branch comparisons
- **Feature walkthroughs** — explain how a feature works by reading source files
- **Architecture overviews** — explore and explain the system structure
- **Setup guides** — document how to set up and run the project
- **Free-form narratives** — tell any story with code, slides, and animations

### `update-movie` — Modify an existing movie

Find an existing movie by ID or title, then:
- Add, edit, reorder, or remove scenes
- Update narration, code blocks, or view types
- Change metadata (title, description)

## What It Does

The skills teach Codex to create and modify code walkthrough videos by calling the merge.mov REST API. Movies can include code views with syntax highlighting, slide views for title cards and diagrams, terminal demos, and custom React animations.

View your movies at [merge.mov](https://merge.mov).

## License

MIT
