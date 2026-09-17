# ReviewForge Java

ReviewForge Java is an automated code-review platform built around a **Java 21 + Spring Boot** backend.

The project analyzes source-code changes, detects common software anti-patterns, stores review results, and integrates with GitHub. The Java backend acts as the main application layer, while a Python/FastAPI service provides code-analysis and optional CodeBERT-based classification.

## Main Technologies

### Java Backend
- Java 21
- Spring Boot 3.3.4
- Spring Security
- Spring Data JPA
- PostgreSQL
- Redis
- Flyway
- Maven
- JWT authentication
- GitHub OAuth and webhooks
- JUnit 5 and Testcontainers

### Supporting Components
- Python + FastAPI — code-analysis worker
- CodeBERT-compatible model pipeline
- Next.js + TypeScript — web dashboard
- GitHub Action — pull-request integration
- VS Code extension — editor integration
- Docker Compose — local deployment

---

## Project Architecture

```text
GitHub / VS Code / Web Dashboard
              |
              v
+--------------------------------+
| Java 21 + Spring Boot Backend  |
| REST API                       |
| Authentication & Security      |
| GitHub Integration             |
| Review Orchestration           |
+---------------+----------------+
                |
        +-------+-------+
        |               |
        v               v
 PostgreSQL + Redis   FastAPI Worker
                          |
                          v
               Rule-based Detection
                    or CodeBERT
```

The **Java Spring Boot application is the main backend** and is located in:

```text
apps/api/
```

It handles API requests, authentication, repository management, GitHub webhooks, persistence, review processing, security, and communication with the code-analysis worker.

---

## Repository Structure

```text
reviewforge-java/
│
├── apps/
│   ├── api/              # Java 21 + Spring Boot backend
│   ├── ml-worker/        # Python/FastAPI analysis service
│   ├── web/              # Next.js dashboard
│   └── vscode-ext/       # VS Code extension
│
├── contracts/            # Shared cross-service test fixtures
├── github-action/        # GitHub Action integration
├── infra/                # Docker Compose configuration
├── scripts/              # Utility/database scripts
├── taxonomy/             # Anti-pattern definitions
├── .env.example          # Example environment configuration
├── render.yaml           # Deployment configuration
└── README.md
```

---

# Running the Project

## Prerequisites

The easiest way to run the complete project is with Docker.

Install:

- Git
- Docker
- Docker Compose v2

For direct Java development, also install:

- JDK 21
- Maven 3.9+

---

## 1. Clone the Repository

```bash
git clone https://github.com/tanmay-alpha/reviewforge-java.git
cd reviewforge-java
```

---

## 2. Create the Environment File

Copy the example configuration:

### Linux / macOS

```bash
cp .env.example .env
```

### Windows PowerShell

```powershell
Copy-Item .env.example .env
```

The supplied configuration is suitable for local development.

For GitHub OAuth or live GitHub repository integration, replace the following values in `.env` with credentials from your own GitHub OAuth application:

```env
GITHUB_CLIENT_ID=change-me
GITHUB_CLIENT_SECRET=change-me
```

Do not commit the `.env` file.

The default project configuration uses:

```env
MODEL_NAME=none
```

so the system can run using the deterministic fallback detector without downloading a model checkpoint.

---

## 3. Start the Complete Application

From the repository root:

```bash
docker compose --env-file .env -f infra/docker-compose.yml up --build
```

Docker Compose starts:

| Service | Address |
|---|---|
| Web Dashboard | http://localhost:3000 |
| Java Spring Boot API | http://localhost:8080 |
| API Health Check | http://localhost:8080/actuator/health |
| FastAPI Worker | http://localhost:8000 |
| ML Worker Health | http://localhost:8000/ml/health |
| PostgreSQL | localhost:5432 |
| Redis | localhost:6379 |

The first build may take several minutes because dependencies and Docker images need to be downloaded.

---

## 4. Verify the Application

Check the Java backend:

```bash
curl http://localhost:8080/actuator/health
```

Expected response:

```json
{"status":"UP"}
```

Check the analysis worker:

```bash
curl http://localhost:8000/ml/health
```

The web dashboard can then be opened at:

```text
http://localhost:3000
```

---

## 5. Stop the Application

```bash
docker compose --env-file .env -f infra/docker-compose.yml down
```

To also remove local Docker volumes:

```bash
docker compose --env-file .env -f infra/docker-compose.yml down -v
```

---

# Java Backend

The main Java project follows the standard Maven/Spring Boot layout:

```text
apps/api/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/com/automatedcodereviewtool/
    │   └── resources/
    └── test/
        ├── java/com/automatedcodereviewtool/
        └── resources/
```

Important Java packages include:

| Package | Purpose |
|---|---|
| `config` | Spring and application configuration |
| `controller` | REST API endpoints |
| `dto` | Request and response objects |
| `entity` | JPA database entities |
| `repository` | Spring Data JPA repositories |
| `service` | Core business logic |
| `security` | JWT, API keys and security filters |
| `webhook` | GitHub webhook processing |
| `exception` | Error handling |

The general backend flow is:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
PostgreSQL
```

For code review:

```text
GitHub
   ↓
Spring Boot API
   ↓
FastAPI Analysis Worker
   ↓
Review Findings
   ↓
PostgreSQL / GitHub
```

---

# Running Java Tests

Java tests can be executed entirely from the command line.

```bash
cd apps/api
mvn -B -ntp test
```

This runs the Java unit, controller, service and integration tests configured for the normal test profile.

For PostgreSQL-specific correctness tests, Docker must be running:

```bash
mvn -B -ntp -Ppostgres-correctness test
```

These tests use Testcontainers with PostgreSQL.

---

# Build the Java API Container

From the repository root:

```bash
docker build -f apps/api/Dockerfile -t reviewforge-java-api .
```

The repository root is used as the Docker build context because the Java tests also use shared fixtures from:

```text
contracts/
```

---

# Other Component Tests

## ML Worker

```bash
cd apps/ml-worker
pip install -r requirements.txt
pytest -m "not slow" -q
```

## Web Dashboard

```bash
cd apps/web
npm install
npm test
npm run build
```

## GitHub Action

```bash
cd github-action
npm install
npm test
npm run build
```

## VS Code Extension

```bash
cd apps/vscode-ext
npm install
npm run compile
npm test
```

---

# Core Features

- GitHub OAuth authentication
- JWT-based user authentication
- API-key authentication
- Repository connection and management
- GitHub webhook verification
- Pull-request diff processing
- Automated code-review workflow
- Anti-pattern detection
- Quality-score generation
- Persistent review results using PostgreSQL
- Redis-backed application state
- Rule-based fallback analysis
- Optional CodeBERT-compatible inference pipeline
- Next.js review dashboard
- GitHub Action integration
- VS Code editor integration

---

# Security

The backend includes:

- Spring Security
- JWT access and refresh tokens
- API-key authentication
- GitHub OAuth state validation
- HMAC verification for GitHub webhooks
- Request-size limits
- Rate limiting
- Repository ownership checks
- Secret redaction before analysis
- Environment-based sensitive configuration

Secrets should never be committed to Git.

Always keep real credentials in `.env` or deployment environment variables.

---

# Environment Configuration

Important variables are documented in `.env.example`.

Examples include:

```env
GITHUB_CLIENT_ID=
GITHUB_CLIENT_SECRET=

JWT_SECRET=
ENCRYPTION_KEY=

ML_WORKER_SECRET=
ML_WORKER_URL=http://localhost:8000

MODEL_NAME=none

NEXT_PUBLIC_API_BASE_URL=http://localhost:8080
FRONTEND_URL=http://localhost:3000
APP_BASE_URL=http://localhost:8080

SPRING_DATASOURCE_URL=
SPRING_DATASOURCE_USERNAME=
SPRING_DATASOURCE_PASSWORD=

REDIS_PASSWORD=
```

For a normal local Docker run, start by copying `.env.example` to `.env`.

---

# Troubleshooting

### `java` or `mvn` is not recognized

Install JDK 21 and Maven, and configure `JAVA_HOME`.

Alternatively, run the complete application using Docker Compose without installing Java locally.

### Port already in use

Make sure ports:

```text
3000
5432
6379
8000
8080
```

are not already occupied by another application.

### GitHub login does not work

Add valid GitHub OAuth credentials to `.env`:

```env
GITHUB_CLIENT_ID=...
GITHUB_CLIENT_SECRET=...
```

### CodeBERT checkpoint is unavailable

The project can operate without a model checkpoint.

Keep:

```env
MODEL_NAME=none
```

to use deterministic fallback detection.

---

# License

This project is licensed under the MIT License.

See [LICENSE](LICENSE) for details.
