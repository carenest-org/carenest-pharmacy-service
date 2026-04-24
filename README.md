# CareNest Pharmacy Service

Pharmacy and prescription management microservice for the CareNest healthcare platform.

---

## 🔄 CI/CD Pipeline

Every push to `main` triggers the full CI/CD pipeline:

```
① SonarQube  →  ② Build  →  ③ Snyk  →  ④ Docker+Trivy+Push  →  ⑤ Approval  →  ⑥ Deploy
```

| Stage | Tool | Purpose | Fail Condition |
|---|---|---|---|
| ① SonarQube | SonarQube | Static code analysis | Quality gate fails |
| ② Build | Node.js 20 | Install dependencies | `npm ci` errors |
| ③ Snyk | Snyk CLI | Dependency vulnerability scan | HIGH/CRITICAL vulns |
| ④ Docker | Docker + Trivy | Build image, scan, push | HIGH/CRITICAL image vulns |
| ⑤ Approval | GitHub Environments | Manual production gate | Reviewer rejects |
| ⑥ Deploy | GitOps (yq) | Update manifest repo | Git push fails |

### Pipeline Behavior
- **Push to `main`**: Full pipeline (CI + CD)
- **Pull request to `main`**: CI only (SonarQube → Build → Snyk)
- **Tag `v*`**: Full pipeline with semantic version tag on Docker image

---

## 🐳 Docker Image Tagging

Every successful pipeline push produces **3 tags**:

```
jayadevarun2003/carenest-pharmacy-service:latest
jayadevarun2003/carenest-pharmacy-service:sha-abc1234
jayadevarun2003/carenest-pharmacy-service:v1.2.0     ← only when git tag exists
```

The `sha-*` tag is used for deployment — it's immutable and traceable to the exact commit.

---

## 🔒 Security Scanning Flow

```
Source Code ──► SonarQube (SAST)
                    │
Dependencies ──► Snyk (SCA)
                    │
Docker Image ──► Trivy (Container Scan)
                    │
              All Pass? ──► Push to DockerHub
```

- **SonarQube**: Checks code quality, bugs, vulnerabilities, code smells
- **Snyk**: Scans `package.json` / `package-lock.json` for known CVEs
- **Trivy**: Scans the final Docker image for OS and library vulnerabilities

---

## 🔐 Required GitHub Secrets

| Secret | Description |
|---|---|
| `SONAR_HOST_URL` | SonarQube server URL |
| `SONAR_TOKEN` | SonarQube authentication token |
| `SNYK_TOKEN` | Snyk API token |
| `DOCKERHUB_USERNAME` | DockerHub username |
| `DOCKERHUB_TOKEN` | DockerHub access token |
| `GH_PAT` | GitHub PAT with repo write access to `carenest-manifest` |

---

## 🛠️ Local Development

```bash
# Install dependencies
npm ci

# Run in development mode (with watch)
npm run dev

# Run in production mode
npm start
```

### Environment Variables
See `.env.example` for required environment variables.

---

## 📦 Docker

```bash
# Build locally
docker build -t carenest-pharmacy-service .

# Run locally
docker run -p 3003:3003 --env-file .env carenest-pharmacy-service
```

The Dockerfile uses a multi-stage build:
1. **Builder stage**: `node:20-alpine` — installs production dependencies
2. **Production stage**: `node:20-alpine` — runs as non-root `appuser`
