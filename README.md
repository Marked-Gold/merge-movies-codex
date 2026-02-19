# Merge Movies — Codex Skill

Generate code walkthrough movies from git diffs using [merge.mov](https://merge.mov).

## Install

```bash
$skill-installer install https://github.com/Marked-Gold/merge-movies-codex
```

## Setup

1. Sign up at [merge.mov](https://merge.mov)
2. Go to [Settings](https://studio.merge.mov/settings) and create an API key
3. Set the environment variable:

```bash
export MERGE_MOVIES_API_KEY="mm_your_key_here"
```

## What It Does

The skill teaches Codex to create code walkthrough videos by:
1. Parsing git diffs to understand code changes
2. Planning scenes that tell a story
3. Writing narration for each scene
4. Calling the merge.mov REST API to build the movie

Movies can include code views with syntax highlighting, slide views for title cards and diagrams, terminal demos, and custom React animations.

View your movies at [studio.merge.mov](https://studio.merge.mov).

## License

MIT
