# FIXES.md

## Fix 1 — Critical: Wrong service URL in docker-compose.yml

**What was wrong:**
`SERVICE_A_URL` was set to `http://localhost:5000` in both `docker-compose.yml` and
as the fallback in `worker.js`.

**Why it's a problem:**
Inside Docker, each service runs in its own network namespace. `localhost` inside
service-b refers to service-b itself, not service-a. The worker would immediately
fail every poll attempt and never successfully reach the API.

**How I fixed it:**
Changed `SERVICE_A_URL=http://localhost:5000` to `SERVICE_A_URL=http://service-a:5000`
in `docker-compose.yml`. Docker Compose creates a shared network and resolves service
names as hostnames automatically.

**What could go wrong if left unfixed:**
`docker-compose up` appears to start successfully but service-b logs nothing but
connection errors. The system silently does not work.

---

## Fix 2 — Security: Hardcoded secrets in docker-compose.yml and app.py

**What was wrong:**
`DB_PASSWORD=admin1234` and `SECRET_KEY=supersecret123` were hardcoded directly in
`docker-compose.yml`. The fallback `"supersecret123"` was also baked into `app.py`
source code.

**Why it's a problem:**
Any developer with repo access (or anyone who ever reads a git log) would have these
credentials. Secrets in source control are a critical vulnerability — they can never
truly be revoked because they exist in git history.

**How I fixed it:**
- Replaced hardcoded values in `docker-compose.yml` with `${DB_PASSWORD}` and
  `${SECRET_KEY}`, loaded from a `.env` file at runtime.
- Added `.env.example` as a template and added `.env` to `.gitignore`.
- Changed `app.py` to raise a `RuntimeError` at startup if `SECRET_KEY` is not set,
  rather than silently falling back to a known-bad value.

**What could go wrong if left unfixed:**
Credentials leaked in version control. The app could run in production with a
publicly known secret key, making session tokens forgeable.

---

## Fix 3 — Security: Hardcoded credentials and insecure SSH in deploy.yml

**What was wrong:**
- Docker Hub password hardcoded as plaintext in the workflow file.
- SSH login as `root` user.
- `StrictHostKeyChecking=no` disables host key verification.
- A hardcoded IP address for the deploy target.

**Why it's a problem:**
Hardcoded credentials in a workflow file are exposed to anyone who can read the repo.
SSH as root gives an attacker full machine access if the key is ever compromised.
`StrictHostKeyChecking=no` opens the connection to man-in-the-middle attacks. A
hardcoded IP makes the pipeline brittle and couples secrets to code.

**How I fixed it:**
- Moved all credentials to GitHub Actions secrets (`DOCKER_USERNAME`, `DOCKER_PASSWORD`,
  `DEPLOY_SSH_KEY`, `DEPLOY_HOST`).
- Changed SSH user from `root` to `deploy` (a restricted deployment user).
- Replaced `StrictHostKeyChecking=no` with `StrictHostKeyChecking=accept-new` which
  trusts on first connect but verifies on subsequent connections.
- Used `--password-stdin` for Docker login to avoid the password appearing in process
  listings.

**What could go wrong if left unfixed:**
Full credentials exposed in source code. A compromised key gives root access to the
production server.

---

## Fix 4 — Functional: service-b never built or deployed in CI/CD

**What was wrong:**
The `deploy.yml` workflow only built and pushed `service-a`. service-b had no build,
push, or deploy step.

**Why it's a problem:**
Any change to service-b would never be deployed. The CI pipeline gives a false sense
of confidence — it passes green while half the system is stale.

**How I fixed it:**
Added a separate build-and-push step for service-b using its own image tag
(`myorg/service-b:${{ github.sha }}`).

**What could go wrong if left unfixed:**
service-b runs a permanently outdated image in production with no way to update it
through the normal deployment pipeline.

---

## Fix 5 — Reliability: depends_on without healthcheck condition

**What was wrong:**
`service-b` used `depends_on: service-a` without a `condition`. This only waits for
the service-a *container to start*, not for the Flask app inside it to be ready to
accept connections.

**Why it's a problem:**
service-b can begin polling before service-a has finished booting, causing failed
requests during the first poll cycle. In slow environments this can cause cascading
failures.

**How I fixed it:**
- Added a `healthcheck` to service-a in `docker-compose.yml` that hits `/health`.
- Changed the `depends_on` for service-b to use `condition: service_healthy`.

**What could go wrong if left unfixed:**
service-b logs connection errors on startup and may miss data from the first polling
window. In stricter retry configurations it could crash-loop.

---

## Fix 6 — Kubernetes: Missing resource limits, probes, and service-b deployment

**What was wrong:**
- `deployment.yaml` defined resource *requests* but no *limits* for service-a.
- No liveness or readiness probes were configured.
- service-b had no Kubernetes Deployment or Service manifest at all.

**Why it's a problem:**
Without limits, a misbehaving container can consume unbounded CPU/memory and starve
other pods on the same node. Without readiness probes, Kubernetes sends traffic to
pods before they are ready. Without a service-b deployment, service-b simply does not
run in the Kubernetes environment.

**How I fixed it:**
- Added `limits` alongside `requests` for service-a.
- Added `livenessProbe` and `readinessProbe` pointing at `/health`.
- Added a full `Deployment` manifest for service-b with appropriate resource limits
  and the correct `SERVICE_A_URL` environment variable pointing to the service-a
  Kubernetes Service DNS name.

**What could go wrong if left unfixed:**
Node resource exhaustion, traffic routed to unready pods, and service-b not running
at all in Kubernetes.

---

## Improvement 1 — Added .env.example and .gitignore entry

Added `.env.example` as a developer-facing template showing which environment
variables are required, with placeholder values. Added `.env` to `.gitignore` to
prevent accidental secret commits. This is standard practice for any project using
environment-based configuration.

---

## Improvement 2 — Used commit SHA image tags in CI instead of `latest`

Changed image tags in the CI pipeline from `:latest` to `${{ github.sha }}`. The
`latest` tag is mutable and makes it impossible to know exactly what code is running
in production, roll back to a previous version, or reproduce a deployment. Using the
commit SHA ties every image to an exact point in git history.