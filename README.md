# CI/CD Pipeline Demo

A Node.js REST API with a production-style, multi-stage GitHub Actions CI/CD pipeline. The pipeline — not the API — is the point: five chained jobs with dependency gates, artifact packaging, and an environment-protected deploy step.

## Pipeline Architecture

```
lint ──► unit-test ──► integration-test ──► build-artifact ──► deploy
```

| Job | What it does | Gate |
|---|---|---|
| `lint` | ESLint code quality check | on every push / PR |
| `unit-test` | Jest tests with coverage report | needs: lint |
| `integration-test` | Playwright API tests against a live server | needs: unit-test |
| `build-artifact` | Packages app into a tarball, uploads as artifact | needs: integration-test |
| `deploy` | Downloads artifact, deploys to staging environment | needs: build-artifact + manual approval |

Each job only runs when the previous one passes (`needs:`). Failing tests block the artifact build; a failed build blocks the deploy. The deploy job targets a GitHub Environment named `staging` — add required reviewers there to require manual sign-off before any production push.

## Tech Stack

| Layer | Tool |
|---|---|
| API | Node.js · Express |
| Unit tests | Jest · Supertest (mock-service pattern) |
| Integration tests | Playwright (API mode) |
| Linting | ESLint 9 |
| CI/CD | GitHub Actions (5-job pipeline) |

## Running Locally

```bash
npm install
npm run lint            # ESLint
npm run test:unit       # Jest unit tests
npm run test:integration # Playwright integration tests
npm start               # API server on http://localhost:3001
```

## API Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/health` | Health check |
| GET | `/api/products` | List all products |
| GET | `/api/products/:id` | Get product by ID |
| POST | `/api/products` | Create a product |
| PUT | `/api/products/:id` | Update a product |
| DELETE | `/api/products/:id` | Delete a product |

**Product schema:**

```json
{
  "id": 1,
  "name": "Widget",
  "category": "Tools",
  "price": 9.99,
  "stock": 50,
  "createdAt": "2025-01-01T00:00:00.000Z"
}
```

## Test Coverage

**Unit tests** (`tests/unit/`):
- `productService.test.js` — create, getAll, getById, update, remove; validation rules; edge cases (33 tests)
- `routes.test.js` — route handlers tested via mock-injected service (mirrors the Moq pattern from C# API testing); 200/201/204/400/404 paths (17 tests)

**Integration tests** (`tests/integration/`):
- Full CRUD flow against a live Express server
- Validation error responses (missing name, negative price, fractional stock)
- 404 handling for unknown IDs
- Response header checks (Content-Type, X-Powered-By absent)

## Key Pipeline Concepts Demonstrated

- **Job dependencies** (`needs:`) — strict sequencing; no deploy without green tests
- **Artifact upload/download** — build job packages the app; deploy job downloads and uses it
- **Coverage publishing** — Jest lcov coverage uploaded as a workflow artifact
- **Test report publishing** — Playwright HTML report uploaded on pass or fail (`if: always()`)
- **Environment protection** — `environment: staging` supports required reviewers and branch policies
- **Conditional deploy** — deploy job only runs on `main` branch pushes, not PRs

## Project Structure

```
cicd-pipeline-demo/
├── src/
│   ├── services/
│   │   └── productService.js     # Business logic + in-memory store
│   ├── routes/
│   │   └── products.js           # Express route factory (dependency-injected)
│   └── middleware/
│       └── errorHandler.js
├── server.js                     # Entry point (separate from app for testability)
├── tests/
│   ├── unit/
│   │   ├── productService.test.js
│   │   └── routes.test.js
│   └── integration/
│       └── products.spec.js
├── .github/
│   └── workflows/
│       └── ci-cd.yml             # 5-job pipeline
├── jest.config.cjs
├── playwright.config.js
└── eslint.config.js
```
