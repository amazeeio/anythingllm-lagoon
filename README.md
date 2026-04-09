# AnythingLLM for Lagoon

This repository provides the Lagoon-facing deployment for [AnythingLLM](https://github.com/Mintplex-Labs/anything-llm). It uses the published `anythingllm-lagoon-base` image so deployments can reuse a shared runtime image from GHCR instead of rebuilding the same application layer on every deploy.

The reusable runtime image is maintained in the `anythingllm-lagoon-base` repository and published as `ghcr.io/amazeeio/anythingllm-lagoon-base:latest`.

## What this repository does

- Pulls the published base image in Lagoon and local Docker Compose
- Supplies deployment-specific environment variables and persistent volume mounts
- Keeps the Lagoon-facing service definition stable for downstream consumers

## Relationship to the base repo

`anythingllm-lagoon-base` owns the shared runtime image used by this repository. That includes:

- The pinned upstream AnythingLLM image version
- Lagoon-compatible file permissions for arbitrary runtime UIDs
- Build-time Prisma client generation
- The custom entrypoint that preserves the current startup behavior

If you need to change the runtime image itself, make that change in `anythingllm-lagoon-base`, publish a new image, and then update this repository if needed.

## Prerequisites

- A Lagoon account with access to create repositories
- Access to an LLM API provider such as amazee.ai
- Access to the published image `ghcr.io/amazeeio/anythingllm-lagoon-base:latest`

## Lagoon deployment

### 1. Add the repository to Lagoon

Add this repository as a Lagoon project. Lagoon will pull the published base image defined in `docker-compose.yml`.

### 2. Configure environment variables

Before deploying, configure these variables in your Lagoon project:

| Variable | Description | Example |
|----------|-------------|---------|
| `JWT_SECRET` | Secret used for AnythingLLM authentication | `replace-with-long-random-value` |
| `LLM_URL` | Base URL for your Generic OpenAI-compatible provider | `https://llm.us103.amazee.ai` |
| `LLM_AI_KEY` | API key for the configured provider | `your-api-key-here` |
| `EMBEDDING_PROVIDER` | Embedding backend | `native` |

Optional database variables when using external Postgres:

| Variable | Description |
|----------|-------------|
| `DB_HOST` | Database host |
| `DB_USER` | Database user |
| `DB_PASS` | Database password |
| `DB_NAME` | Database name |
| `DB_PORT` | Database port |

This repository maps `LLM_URL` and `LLM_AI_KEY` onto the internal AnythingLLM `OPEN_AI_BASE_PATH` and `OPEN_AI_KEY` variables for you.

### 3. Deploy

Deploy the project. Lagoon will pull the published image and start AnythingLLM.

### 4. Complete first-run setup

After deployment:

1. Open the Lagoon-provided route
2. Complete the onboarding wizard
3. Confirm the LLM provider settings if you did not set them entirely through environment variables
4. Create the admin account
5. Upload content and begin using the workspace

## Local development

Local development uses the same published image as Lagoon.

### 1. Start the service

```bash
docker compose pull
docker compose up -d
```

The UI will be available at `http://localhost:3000`.

### 2. View logs

```bash
docker compose logs -f anythingllm
```

### 3. Connect to the container

```bash
docker compose exec anythingllm bash
```

### 4. Stop the service

```bash
docker compose down
```

## Troubleshooting

### Check Lagoon logs

```bash
lagoon logs -p <project-name> -e prod
```

### Verify persistent storage

```bash
lagoon ssh -p <project-name> -e prod
ls -la /app/server/storage
```

### LLM connection issues

1. Check that `LLM_URL` and `LLM_AI_KEY` are set correctly
2. Confirm the provider is reachable from the deployed environment
3. Review AnythingLLM logs for provider-specific errors

### Document upload failures

1. Check that the persistent volume has free space
2. Verify the uploaded file format is supported
3. Review AnythingLLM logs for processing failures

## Notes

- Runtime state is stored in Lagoon at `/app/server/storage`
- The service runs as UID `10000` in local Compose to match Lagoon expectations
- The shared image preserves the current custom startup behavior, including skipping runtime `prisma generate`

## Resources

- [AnythingLLM Documentation](https://docs.useanything.com/)
- [Lagoon Documentation](https://docs.amazee.io/lagoon-documentation/)
