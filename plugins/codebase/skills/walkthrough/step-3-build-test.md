# Step 3: Build & Test

**Goal:** Understand how to build, run, and test this project.

Examine `Makefile`, `package.json` scripts, CI config (`.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`), `Dockerfile`, `docker-compose.yml`, and README.

## Output

```markdown
## Build & Test

### Build

- **Command:** `make build` / `go build ./cmd/server` / `npm run build`
- **Artifacts:** binary at `./bin/server`, or bundle at `./dist/`
- **Prerequisites:** Go 1.21+, protoc, Node 18+

### Run

- **Local:** `make run` / `go run ./cmd/server`
- **Docker:** `docker-compose up`
- **Required env vars:** `DB_HOST`, `REDIS_URL`, `KAFKA_BROKERS`

### Test

- **Unit tests:** `make test` / `go test ./...` / `npm test`
- **Integration tests:** `make test-integration` (requires Docker)
- **Linting:** `make lint` / `golangci-lint run`

### CI/CD

- <describe pipeline stages if CI config exists>
```
