# Setup

## Get an API Key

1. Sign up at [merge.mov](https://merge.mov)
2. Go to [Settings](https://studio.merge.mov/settings)
3. Create a new API key

## Configure the Environment

Set the API key as an environment variable:

```bash
export MERGE_MOVIES_API_KEY="mm_your_key_here"
```

## Verify

```bash
curl -s "https://merge.mov/api/movies" \
  -H "X-API-Key: $MERGE_MOVIES_API_KEY"
```

Should return a JSON array of movies (empty `[]` for new accounts).
