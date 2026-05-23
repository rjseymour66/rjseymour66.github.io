+++
title = 'Production'
date = '2026-05-23T10:19:48-04:00'
weight = 100
draft = false
+++

## Production complexity

Moving code from development to production reveals a new class of problems. Development environments are controlled and predictable; production is not.

### Users are unpredictable

Users introduce complexity that is difficult to simulate during development. Five factors make production behavior hard to anticipate:

- *Real-world data*: production data is messier, more varied, and larger than test data. Edge cases that never appear in development surface immediately when real users interact with the system.
- *Scale and concurrency*: hundreds or thousands of simultaneous users create race conditions, lock contention, and resource exhaustion that are nearly impossible to reproduce in a single-developer environment.
- *Diverse environments*: users run different browsers, operating systems, network speeds, and device capabilities. A feature that works on a fast laptop may fail on a slow mobile connection.
- *Environmental drift*: production dependencies, OS patches, and third-party services change over time. A deployment that worked last month may behave differently today due to changes outside your control.
- *Unexpected use cases*: users find ways to use your application that you never anticipated. They chain requests, use features in unintended sequences, and push edge cases you did not consider.

You can prepare for unpredictability with three practices. *Canary testing* routes a small percentage of production traffic to a new version before a full rollout, limiting the blast radius of any regression. *A/B testing* exposes different behaviors to different user segments, letting you measure the impact of a change before committing to it. *Robust observability* gives you the visibility to detect and diagnose unexpected behavior quickly when it occurs.

### Unable to reproduce

Production bugs are difficult to reproduce in development because many of the conditions that trigger them do not exist outside of production. Six categories of variables change between environments:

- *Environment differences*: operating system versions, library versions, file system layouts, and network topology all differ between a developer's machine and a production server. A bug triggered by a specific library version may never appear in development.
- *Configuration management*: database connection strings, feature flags, cache TTLs, and service endpoints differ in each environment. A misconfigured timeout in production that does not exist locally can cause intermittent failures that are impossible to reproduce.
- *Concurrency and load*: a race condition that requires thousands of concurrent requests to trigger will never appear in local testing. Many production bugs only manifest under sustained load.
- *Data diversity*: production databases contain years of real data with edge cases, encoding inconsistencies, and legacy formats. A query that runs in milliseconds against a clean development dataset may time out against a table with hundreds of millions of rows.
- *External dependencies*: third-party APIs, payment processors, and cloud services behave differently in production than in sandbox environments. Rate limits and service-level agreements do not apply in development.
- *Security constraints*: production environments enforce network policies, firewall rules, and IAM permissions that do not exist in development. A service that calls another freely in development may be blocked by a security group rule in production.

Four practices close the gap between environments. *Containerization* packages your application with its dependencies, making the runtime environment reproducible across development, staging, and production. *Observability* gives you logs, metrics, and traces in production so you can diagnose issues with real data rather than guessing. *Environmental parity* means staging mirrors production as closely as possible, including data volume, configuration, and infrastructure. *CI/CD* automates the path from code change to production deployment, reducing the manual steps where environment differences are typically introduced.

## Building production ready code

Production-ready code goes beyond functional correctness. You need to consider performance, environment-specific configuration, error handling and logging, and security.

### Performance optimization

Performance optimization improves the experience for your users and reduces infrastructure costs. Apply optimizations where profiling shows they matter rather than guessing.

- *Asynchronous programming*: offload long-running operations like database queries, file I/O, and external API calls to goroutines so your application handles more requests concurrently without blocking.

```go
func handleOrder(ctx context.Context, w http.ResponseWriter, r *http.Request) {
    order, err := parseOrder(r)
    if err != nil {
        http.Error(w, "invalid order", http.StatusBadRequest)
        return
    }
    w.WriteHeader(http.StatusAccepted)
    go func() {
        if err := processOrder(context.WithoutCancel(ctx), order); err != nil {
            log.Printf("order processing failed: %v", err)
        }
    }()
}
```

- *Reducing network calls*: each network call adds latency. Batch requests where possible, cache responses for data that does not change frequently, and colocate services that communicate heavily.
- *Caching strategies*: store the results of expensive operations in memory so subsequent requests return immediately. See the caching strategies section in Storing Data for implementation patterns.
- *Database query optimization*: use indexes, avoid `SELECT *`, and use `EXPLAIN ANALYZE` to identify slow queries. See the querying and managing data performance section for details.
- *Code minification and bundling*: for web applications, minification removes whitespace and comments from JavaScript and CSS, and bundling combines multiple files into one. Both reduce the number and size of files the browser must download.
- *Lazy loading*: defer loading resources until they are needed. Load images as the user scrolls rather than all at once. Lazy loading reduces initial page load time and avoids loading resources the user never reaches.
- *Content delivery networks (CDNs)*: CDNs distribute static assets across servers geographically close to your users. A user in Tokyo served from a CDN node in Tokyo receives assets far faster than from a server in Virginia.

### Environment specific configurations

Your application needs tailored configurations to perform reliably in each environment. A configuration appropriate for local development may be wrong or dangerous in production.

Common configuration values that differ between environments:

- *Logging*: development uses verbose debug logging; production uses structured logs at INFO or WARN level to reduce noise and storage costs.
- *External service endpoints*: development points to sandbox APIs and mock services; production points to live services with real data and real costs.
- *Database settings*: connection pool sizes, query timeouts, and replica routing differ between a local database and a production cluster.
- *Cache settings*: local development may have no cache; production uses Redis with tuned TTLs and eviction policies.
- *Email*: development sends to a local mail trap or logs to console; production sends through a transactional email provider.
- *Security*: HTTPS is optional locally and required in production. CORS policies, rate limits, and cookie security flags all differ.
- *Feature flags and application behavior*: experimental features may be enabled in staging and disabled in production until they are ready.

### Configuration files

Configuration files let you define a separate set of values for each environment, loaded at startup based on the current environment:

```yaml
# config.production.yaml
database:
  host: db.internal.example.com
  port: 5432
  max_open_conns: 25
  max_idle_conns: 25

cache:
  host: redis.internal.example.com
  ttl: 5m

logging:
  level: info
  format: json
```

The application reads the file matching the current environment, set by an environment variable like `APP_ENV=production`.

### Environment variables

Configuration files pose two risks: they may check sensitive values into version control, and they do not support credential rotation without a file change. Environment variables solve both problems.

Inject secrets at runtime by referencing environment variables from your configuration file:

```yaml
database:
  host: ${DB_HOST}
  port: ${DB_PORT}
  password: ${DB_PASSWORD}

cache:
  host: ${REDIS_HOST}

email:
  api_key: ${EMAIL_API_KEY}
```

The application reads `DB_PASSWORD` from the environment at startup. The configuration file itself contains no secrets and is safe to commit to version control.

### Feature flags

*Feature flags* let developers enable or disable features at runtime without deploying new code. They act as a control panel for your application's features, letting you ship code to production while keeping it hidden from users until it is ready.

Three common types of feature flags:

- *Release flags*: hide an incomplete feature from all users while development continues. Flip the flag when the feature is ready.
- *Experiment flags*: expose a feature to a subset of users to measure its impact before a full rollout. Used for A/B testing and gradual releases.
- *Permission flags*: control access to features based on user role, subscription tier, or other attributes.

```go
type FeatureFlags struct {
    NewCheckoutFlow bool
    BetaDashboard   bool
}

func LoadFlags() *FeatureFlags {
    return &FeatureFlags{
        NewCheckoutFlow: os.Getenv("FLAG_NEW_CHECKOUT") == "true",
        BetaDashboard:   os.Getenv("FLAG_BETA_DASHBOARD") == "true",
    }
}

func handleCheckout(flags *FeatureFlags, w http.ResponseWriter, r *http.Request) {
    if flags.NewCheckoutFlow {
        newCheckoutHandler(w, r)
        return
    }
    legacyCheckoutHandler(w, r)
}
```

For more sophisticated use cases, feature flag services like LaunchDarkly or Unleash provide targeting rules, percentage rollouts, and real-time updates without redeployment.

### Secrets

*Secrets* are sensitive configuration values — API keys, passwords, tokens, and certificates — that require special handling.

Never store secrets directly in code or commit them to version control. If a secret is committed, assume it is compromised. Removing it from the current commit is not enough: the secret exists in git history and must be purged using a tool like `git filter-repo`.

Best practices:

- Use a dedicated secrets management service such as AWS Secrets Manager, HashiCorp Vault, or Azure Key Vault. These services provide access control, audit logging, and automatic rotation.
- Rotate credentials regularly. Make rotation a standard practice rather than a response to a breach.
- Apply the principle of least privilege: each service should have access only to the secrets it needs.
- Development, staging, and production should never share credentials. Shared credentials mean a compromised development environment can expose production.

## Error handling and logging

Handling edge cases and errors often accounts for 80% of your development time and effort. Production systems encounter conditions that development environments never do: network timeouts, malformed input, third-party failures, and resource exhaustion.

#### Error handling

- *Graceful degradation*: when a non-critical component fails, the application continues to function with reduced capability rather than failing entirely. If a recommendation service is unavailable, the application still serves the page without recommendations.

```go
func getRecommendations(ctx context.Context, userID int64) []Product {
    products, err := recommendationService.Fetch(ctx, userID)
    if err != nil {
        log.Printf("recommendations unavailable for user %d: %v", userID, err)
        return nil
    }
    return products
}
```

- *Meaningful error messages*: errors returned to users should describe what went wrong without exposing internal details. Errors logged internally should include context: the operation that failed, the input that triggered it, and the underlying cause.

```go
if err := db.CreateUser(ctx, user); err != nil {
    log.Printf("createUser: failed to insert user %s: %v", user.Email, err)
    return fmt.Errorf("createUser: %w", err)
}
```

- *Logging for debugging*: log errors at the point they are handled, not at every layer they pass through. Include request IDs or trace IDs so you can correlate log entries across services.
- *Fallback options*: when a primary resource is unavailable, fall back to a secondary. Return a cached stale value rather than making the user wait on a slow database.

#### Logging

Use structured logging with consistent log levels so log aggregation tools can filter and alert on specific conditions.

Log levels from highest to lowest severity:

- `ERROR`: something failed and requires immediate attention. The operation did not complete successfully.
- `WARN`: something unexpected occurred but the operation completed. May indicate a degrading condition.
- `INFO`: normal operational events such as startup, shutdown, and significant state changes.
- `DEBUG`: detailed diagnostic information useful during development. Disable in production.

Do not log:

- PII or sensitive information: names, email addresses, tokens, IP addresses, or SSNs
- Large objects or entire response payloads
- Duplicate entries of the same event across multiple layers

## Security essentials

Security is not a feature you add at the end. Build it into every layer of your application from the start. The Open Web Application Security Project (OWASP) maintains a regularly updated list of the most critical web application security risks at owasp.org.

### Secure communications with HTTPS

HTTPS is required for production. HTTP transfers data as plain text, exposing passwords, tokens, and personal information to anyone on the network path. HTTPS adds a layer of encryption using TLS (Transport Layer Security).

TLS certificates act as digital identity documents for servers. A server presents its certificate to prove its identity. Certificates are issued by trusted third parties called Certificate Authorities (CAs). After your browser verifies the certificate against a trusted CA, TLS uses the established trust to set up strong encryption for all data in transit.

Best practices:

- Use HTTPS for every request, including internal service-to-service communication.
- Redirect all HTTP requests to HTTPS automatically.
- Keep certificates current. Expired certificates break your application and erode user trust.
- Use TLS 1.2 or higher. Older versions (SSL 3.0, TLS 1.0, TLS 1.1) have known vulnerabilities that allow attackers to decrypt traffic even when encryption appears to be in use.

### Authentication best practices

*Authentication* verifies that users are who they claim to be. *Authorization* determines what verified users are allowed to do. The most important rule: never implement your own authentication or cryptographic primitives. Use established libraries and services maintained by security experts.

The building blocks of a robust authentication system:

- *Strong password management*: never store passwords in plain text. Hash passwords using a slow, purpose-built algorithm like bcrypt or Argon2, and always use a unique salt per user to prevent rainbow table attacks.
- *Securing user accounts*: introduce delays after failed login attempts to slow brute force attacks. Monitor for repeated failures and lock or flag the account. Add a CAPTCHA after a threshold of failures.
- *Effective session management*: use cryptographically secure tokens with sufficient entropy so sessions cannot be guessed or hijacked. Set idle timeouts (end the session after a period of inactivity) and absolute timeouts (end the session after a maximum duration regardless of activity).
- *Multi-factor verification*: add a second layer of verification beyond the password, such as a TOTP code or hardware key.
- *Behavioral triggers*: trigger additional verification when unusual behavior is detected, such as a login from a new device or location.

#### Safeguarding user data

*Personally identifiable information* (PII) is any data that can identify a specific individual. PII includes obvious data like names and Social Security numbers, but also IP addresses, device identifiers, and behavioral data that can identify someone when combined.

Mark sensitive fields explicitly in your data models so they are easy to identify during logging and serialization reviews:

```go
type User struct {
    ID        int64     `json:"id"`
    Email     string    `json:"email"  sensitive:"true"`
    Phone     string    `json:"phone"  sensitive:"true"`
    Password  string    `json:"-"`
    CreatedAt time.Time `json:"created_at"`
}
```

The `json:"-"` tag prevents the password field from ever appearing in serialized output. The `sensitive:"true"` tag marks fields for redaction in logs and error messages. Collect only the data you need. Every piece of PII you store is a liability. Data you do not collect cannot be breached.

#### Encryption

Encryption should be your last line of defense, not your only one. If an attacker reaches your data, strong encryption is what prevents them from reading it.

The *Advanced Encryption Standard* (AES) is the current standard for symmetric encryption. Choose a key length based on the sensitivity of the data:

- *AES-128*: suitable for most applications. Provides strong protection against brute force attacks for the foreseeable future.
- *AES-256*: required for highly sensitive data or long-lived secrets. Provides a higher security margin at a slight CPU cost.

Always use AES in GCM (Galois/Counter Mode), which provides both encryption and authentication. Never use ECB mode, which does not protect against pattern analysis.

```go
func encrypt(key, plaintext []byte) ([]byte, error) {
    block, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    }
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, err
    }
    nonce := make([]byte, gcm.NonceSize())
    if _, err := io.ReadFull(rand.Reader, nonce); err != nil {
        return nil, err
    }
    return gcm.Seal(nonce, nonce, plaintext, nil), nil
}

func decrypt(key, ciphertext []byte) ([]byte, error) {
    block, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    }
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, err
    }
    nonceSize := gcm.NonceSize()
    if len(ciphertext) < nonceSize {
        return nil, errors.New("ciphertext too short")
    }
    nonce, ciphertext := ciphertext[:nonceSize], ciphertext[nonceSize:]
    return gcm.Open(nil, nonce, ciphertext, nil)
}
```

#### Compliance requirements

Compliance means following specific rules, standards, and regulations that govern how you handle data. Requirements vary by industry and geography. Implement encryption for all data in transit (TLS) and at rest (AES) as a baseline for any compliance framework.

Key privacy regulations:

- *GDPR* (General Data Protection Regulation): European Union law governing how personal data of EU residents is collected, stored, and processed. Requires explicit consent, the right to be forgotten, and data breach notification within 72 hours.
- *CCPA* (California Consumer Privacy Act): California law giving residents the right to know what data is collected about them, the right to delete it, and the right to opt out of its sale.
- *HIPAA* (Health Insurance Portability and Accountability Act): US law governing the handling of protected health information (PHI). Requires strict access controls, audit logging, and breach notification for healthcare applications.
- *PCI DSS* (Payment Card Industry Data Security Standard): security standards for applications that handle cardholder data. Requires encryption, network segmentation, access controls, and regular security testing.
- *SBOM* (Software Bill of Materials): an inventory of all software components and dependencies in your application. Required by US executive orders for federal software and increasingly expected by enterprise customers to identify supply chain vulnerabilities.

## Deployment pipeline

A deployment pipeline is your roadmap for safely and reliably moving code from development to production. Four interconnected components work together to make this possible:

- *Deployment environments*: the distinct places where your code lives and runs at each stage of its journey, from a developer's laptop to production servers.
- *Version control strategies*: the workflows that let multiple developers contribute changes to the same codebase without overwriting each other's work.
- *Deployment automation*: the scripts and tooling that move code between environments consistently and without manual steps.
- *CI/CD*: ties everything together by automatically moving code through the pipeline whenever a change is pushed, running tests and checks at each stage.

## Deployment environments

Most applications move through a sequence of environments before reaching users. Each serves a distinct purpose.

- *Local development*: a developer's own machine. Fast iteration, no shared state, and no risk of affecting other developers or users. Dependencies are often mocked or run locally in containers.
- *Testing / QA*: a shared environment where automated tests and manual quality assurance run against integrated code. It mirrors production infrastructure more closely than local development and exposes integration bugs that only appear when services talk to each other.
- *Staging*: the closest environment to production. It uses production-equivalent infrastructure, configuration, and data (anonymized where required). Staging is where final validation happens before a release. If it does not pass staging, it does not go to production.
- *Production*: the live environment serving real users. Changes here have real consequences. Deployments to production should be deliberate, tested, and reversible.

Each environment requires its own configuration. Database credentials, API endpoints, logging levels, and feature flags must be set correctly for the environment they run in. A misconfigured staging environment that points at a production database is not a staging environment.

## Version control strategies

Version control solves several problems that arise when multiple developers work on the same codebase: conflicting changes that overwrite each other's work, no record of why a change was made, no way to revert a bad change, and no safe place to develop a feature without disrupting others.

Four popular workflows address these problems in different ways:

- *Git Flow*: uses two long-running branches (`main` and `develop`) and a set of short-lived supporting branches for features, releases, and hotfixes. Well-suited to projects with scheduled releases and multiple versions in active maintenance.
- *GitHub Flow*: a simpler model with a single long-running branch (`main`) and short-lived feature branches that merge directly to `main` through pull requests. Well-suited to teams that deploy continuously.
- *Trunk-based development*: developers commit directly to a single shared branch (the trunk) in small, frequent increments. Feature flags hide incomplete work. Requires strong automated testing and a mature CI pipeline.
- *Release train*: code is merged to a release branch on a fixed schedule regardless of individual feature readiness. Teams that miss the train wait for the next one. Useful for large organizations coordinating many teams across a predictable release calendar.

### Core branches

Git Flow is built around two long-running branches that form the foundation of the project.

*`main`* contains production-ready code. Every commit on `main` represents a version that has been tested and is safe to deploy. Protect it by:

- blocking direct commits and requiring all changes to arrive through pull requests
- requiring all automated tests to pass before a merge is allowed
- requiring at least one code review approval

*`develop`* is the shared integration workspace where completed features accumulate before a release. It should always be in a buildable state. Protect it by:

- requiring code review before any branch merges in
- running the full test suite on every merge
- keeping it deployable to the testing environment at all times

### Supporting branches

Git Flow uses short-lived branches for specific tasks. They are created from a defined source branch and merge back to a defined target when the task is complete.

*Feature branches* isolate work on a single feature or fix. Use a consistent naming convention so branches are easy to identify: `feature/user-authentication`, `feature/payment-api`, `fix/login-timeout`. Create from `develop`, merge back to `develop` when complete.

*Release branches* solve the problem of stabilizing a release while development continues. When a release is ready to prepare, a release branch is cut from `develop`. This creates a dedicated stabilization period where only bug fixes, documentation, and release preparation work is allowed. The rest of the team continues merging new features into `develop` without blocking or destabilizing the release. Quality gates and final checks run on the release branch. When it is ready, it merges into both `main` (tagged with the version number) and back into `develop` to carry forward any fixes made during stabilization.

*Hotfix branches* let you patch production without touching unstable code. When a critical bug is found in production, a hotfix branch is cut directly from `main`. This keeps the fix isolated from any work-in-progress on `develop` or a release branch, maintains a clean history, and lets you move quickly and with focus. When complete, the hotfix merges into both `main` and `develop`.

```
 main        ──●───────────────────────────●────────●──▶
               │                           ↑        ↑
 hotfix        │                      ●───●╯        │
               │                      ↑             │
 release       │              ●───●───●─────────────╯
               │              ↑   (stabilize)
 develop  ──●──●──●───●───●───●──────────────────────▶
                   ↑       ↑
 feature      ●───●╯  ●───●╯
```

## Deployment automation

Deployment automation replaces manual steps with repeatable, auditable scripts. A well-automated deployment covers code preparation and packaging, testing and verification, server preparation, deployment execution, post-deployment verification, and rollback capability.

### Scripting deployments

Start with a single script that captures the exact steps your deployment requires. The script below handles a typical application deployment:

```bash
#!/usr/bin/env bash
set -euo pipefail

# Configuration
APP_NAME="myapp"
DEPLOY_DIR="/opt/${APP_NAME}"
BACKUP_DIR="/opt/${APP_NAME}-backups"
ARTIFACT="${1:?Usage: deploy.sh <artifact-path>}"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Create directories
mkdir -p "${DEPLOY_DIR}" "${BACKUP_DIR}"

# Back up the current deployment
if [[ -d "${DEPLOY_DIR}/current" ]]; then
    echo "Backing up current deployment..."
    cp -r "${DEPLOY_DIR}/current" "${BACKUP_DIR}/${TIMESTAMP}"
fi

# Stop the running application
echo "Stopping ${APP_NAME}..."
systemctl stop "${APP_NAME}" || true

# Deploy the new version
echo "Deploying new version..."
mkdir -p "${DEPLOY_DIR}/current"
tar -xzf "${ARTIFACT}" -C "${DEPLOY_DIR}/current"

# Start the application
echo "Starting ${APP_NAME}..."
systemctl start "${APP_NAME}"

# Health check
echo "Running health check..."
sleep 3
if systemctl is-active --quiet "${APP_NAME}"; then
    echo "Deployment successful."
else
    echo "Health check failed. Check logs: journalctl -u ${APP_NAME} -n 50"
    exit 1
fi
```

`set -euo pipefail` makes the script exit immediately on any error, treats unset variables as errors, and catches failures in pipes. These three flags prevent a failed step from being silently ignored.

### Rollback procedures

Every deployment needs a plan to quickly revert changes. Write and test your rollback script before you need it.

```bash
#!/usr/bin/env bash
set -euo pipefail

# Configuration
APP_NAME="myapp"
DEPLOY_DIR="/opt/${APP_NAME}"
BACKUP_DIR="/opt/${APP_NAME}-backups"

# Find the most recent backup
LATEST_BACKUP=$(ls -td "${BACKUP_DIR}"/*/ 2>/dev/null | head -1)
if [[ -z "${LATEST_BACKUP}" ]]; then
    echo "No backups found in ${BACKUP_DIR}. Cannot roll back."
    exit 1
fi
echo "Rolling back to: ${LATEST_BACKUP}"

# Stop the current application
echo "Stopping ${APP_NAME}..."
systemctl stop "${APP_NAME}" || true

# Restore the backup
echo "Restoring backup..."
rm -rf "${DEPLOY_DIR}/current"
cp -r "${LATEST_BACKUP}" "${DEPLOY_DIR}/current"

# Start the application
echo "Starting ${APP_NAME}..."
systemctl start "${APP_NAME}"

# Health check
echo "Running health check..."
sleep 3
if systemctl is-active --quiet "${APP_NAME}"; then
    echo "Rollback successful."
else
    echo "Health check failed after rollback. Check logs: journalctl -u ${APP_NAME} -n 50"
    exit 1
fi
```

Prerequisites for a reliable rollback:

- Verify that backup versions exist before every deployment
- Test the rollback script regularly in staging, not for the first time during an incident
- Document database migration rollback steps separately — application rollback and schema rollback are not the same operation
- Keep multiple backup versions so you can roll back more than one deployment if needed
- Monitor the system during and after rollback to confirm the previous version is healthy

Start with a simple script and build complexity gradually as your deployment process matures.

## Deployment strategies

How you move a new version into production determines your risk exposure and your ability to recover from a bad release.

*All-at-once (big bang)*: the new version replaces the old version in a single operation. Simple to execute and reason about, but carries the highest risk. If the new version has a critical defect, every user is affected immediately and the window between deploy and detection determines how much damage is done. Appropriate for small applications, internal tools, or systems where a brief outage is acceptable.

*Gradual deployment (phased)*: the new version is introduced incrementally, giving you time to detect problems before they affect all users.

- *Canary deployments*: route a small percentage of traffic (typically 1–5%) to the new version while the rest continues on the old version. Monitor error rates, latency, and business metrics for the canary slice. If the metrics look healthy, gradually increase the percentage until the rollout is complete. If something goes wrong, redirect all traffic back to the old version.
- *Rolling deployments*: replace instances of the old version one at a time or in small batches. At any point during the rollout, both versions are running simultaneously. Rolling deployments require your application to handle requests from both versions of the API during the transition.

*Zero-downtime deployments (blue-green)*: maintain two identical production environments, blue and green. One is live and serving all traffic; the other is idle. Deploy the new version to the idle environment, run your validation suite against it, then switch the load balancer to send all traffic to the new environment. The old environment stays up as an instant rollback target. If the new version has a problem, flip the load balancer back. Blue-green deployments eliminate deployment downtime and make rollback nearly instantaneous, at the cost of running duplicate infrastructure.

## Continuous integration and continuous deployment

*Continuous integration* (CI) automatically combines and tests code changes from the entire development team whenever a developer pushes. *Continuous deployment* (CD) takes tested code and automatically releases it to production. Together, CI/CD removes the manual, error-prone steps between writing code and delivering it to users.

The benefits:

- *Consistency*: every change goes through the same automated process. Human judgment is removed from the mechanical steps of building, testing, and packaging, eliminating the "it worked on my machine" class of problems.
- *Automated tests*: the pipeline runs your full test suite on every change. Tests that would be skipped under deadline pressure run automatically, every time.
- *Reduced stress*: small, frequent releases are less risky than large, infrequent ones. When you ship ten changes a day, each individual change is small and easy to reason about. Deployments stop being events and become routine.
- *Early detection of integration issues*: CI surfaces conflicts between branches quickly, when they are cheap to fix, rather than during a stressful pre-release integration phase.
- *Reduced time between code and production*: automated pipelines compress the cycle from merged code to running in production from days or weeks to minutes or hours. Feedback from real users reaches developers faster.

### Building a basic CI/CD workflow

A typical pipeline moves code through five stages:

1. *Code*: a developer pushes a commit or opens a pull request. This event triggers the pipeline.
2. *Build*: the application is compiled. A failed build surfaces integration errors immediately.
3. *Test*: automated tests run against the built artifact. Unit tests, integration tests, and linters all run here.
4. *Package*: the tested application is bundled into a deployable artifact — a binary, a container image, or an archive.
5. *Deploy*: the artifact is deployed to the target environment. On a pull request this may be a staging environment; on a merge to `main` it may be production.

The following GitHub Actions workflow implements this pipeline for a Go application:

```yaml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-test-package:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: '1.22'

      - name: Build
        run: go build -v ./...

      - name: Test
        run: go test -race -coverprofile=coverage.out ./...

      - name: Package
        run: |
          go build -o bin/app ./cmd/app
          tar -czf app-${{ github.sha }}.tar.gz bin/

      - name: Store artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-${{ github.sha }}
          path: app-${{ github.sha }}.tar.gz
          retention-days: 7
```

The `-race` flag enables Go's race detector during tests. The artifact is named with the commit SHA so every build is uniquely identifiable and traceable back to the exact code that produced it.

### Advanced CI/CD patterns

*Canary releases*: the pipeline deploys the new version to a small slice of production infrastructure rather than all of it. The CI/CD system monitors error rates and latency for the canary slice. If metrics stay within defined thresholds, the pipeline automatically increases the canary percentage until the rollout is complete. If a threshold is breached, the pipeline automatically rolls back without human intervention.

*Blue-green deployments*: the pipeline deploys to the idle environment, runs a smoke test suite against it, then updates the load balancer to shift traffic. The switch is a single configuration change, making it fast and reversible. The pipeline can automatically roll back by flipping the load balancer back if post-deploy health checks fail.

## Production system monitoring and maintenance

Deploying code is not the end of your responsibility. Production systems degrade over time: dependencies accumulate vulnerabilities, performance erodes under growing data volumes, and failures occur in ways that development never predicted. Monitoring tells you when something is wrong; maintenance keeps the system healthy before problems occur.

### Monitoring

Monitoring and logging give you visibility into the state of your application. Without them, you learn about problems from user complaints rather than your own systems.

Two types of monitoring work together:

- *Real-time monitoring* tracks live system metrics: request rate, error rate, response latency, CPU and memory usage, and database connection pool utilization. Real-time dashboards and alerts notify you when a metric crosses a threshold so you can respond before users are significantly affected.
- *Log analysis* captures a detailed record of what the system did. Logs let you reconstruct the sequence of events that led to a failure, identify patterns across many requests, and diagnose issues that metrics alone cannot explain.

Monitoring rules:

- Log the information you will need when problems occur, not just when everything is working.
- Always include timestamps and user or request IDs so you can trace a sequence of events.
- Mark errors as `ERROR`, not `INFO`. Log aggregation tools filter by level; an error buried in info logs will not trigger an alert.
- Keep private data and PII out of logs.
- Rotate and delete old logs before they consume your available storage.

Use AI-powered log analysis tools to search through large log volumes and detect anomalies or patterns that indicate emerging issues before they become critical. Tools like this surface low-frequency errors or gradual degradation trends that manual review would miss.

### System maintenance

Keep your production systems current. Unpatched software is one of the most common vectors for security breaches, and outdated dependencies are a leading source of vulnerabilities in production applications.

#### System updates

Apply patches regularly to address:

- *Critical vulnerabilities*: unpatched CVEs are actively exploited. Apply security patches as soon as they are available for production systems.
- *Performance improvements*: runtime and OS updates often include fixes for memory leaks, scheduler inefficiencies, and I/O bottlenecks that affect your application without any code changes on your part.
- *Bug fixes*: library and runtime bugs can cause intermittent failures that are difficult to diagnose. Staying current reduces the search space when investigating production issues.
- *New features and capabilities*: updated runtimes and tools expand what your application can do and often improve developer productivity.

#### Dependency management

Your application may rely on dozens or hundreds of external libraries, any of which may contain security vulnerabilities. Maintain a Software Bill of Materials (SBOM) to track every dependency and its version.

Automate dependency scanning with a GitHub Actions workflow so vulnerabilities are caught before they reach production:

```yaml
name: Dependency security scan

on:
  push:
    branches: [main]
  schedule:
    - cron: '0 8 * * 1'  # every Monday at 08:00 UTC

jobs:
  scan:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: '1.22'

      - name: Install govulncheck
        run: go install golang.org/x/vuln/cmd/govulncheck@latest

      - name: Run vulnerability scan
        run: govulncheck ./...

      - name: Generate SBOM
        run: |
          go install github.com/anchore/syft/cmd/syft@latest
          syft . -o spdx-json > sbom.json

      - name: Store SBOM
        uses: actions/upload-artifact@v4
        with:
          name: sbom-${{ github.sha }}
          path: sbom.json
          retention-days: 90
```

`govulncheck` is Go's official vulnerability scanner. It checks your module's dependencies against the Go vulnerability database and reports only vulnerabilities that are reachable from your code, reducing noise from libraries you import but do not actually call. The scheduled trigger runs the scan weekly so newly disclosed vulnerabilities are caught even when no code changes are pushed.

## Putting into practice

The concepts in this guide are most useful when they shape your habits at each stage of development. Here is what to do at each stage.

### Before you code

- Write down the top three to five things that could go wrong. For each one, note what error you would surface to the user and what you would log internally.
- Identify every action worth logging: user authentication, state changes, external API calls, and anything that touches money or personal data.
- List every configuration value your feature will need: API endpoints, timeouts, feature flags, credentials. Note which values will differ between your local machine and production.

### While coding

- Create a feature branch with a clear, descriptive name: `feature/order-confirmation-email`, not `feature/fix` or `my-branch`.
- Add error handling for the top three failure cases you identified. You do not need to handle every edge case on the first pass, but the most likely failures should not go unhandled.
- Log key actions as you build them: when a request arrives, when an external call is made, when something fails. It is easier to add logging alongside the code than to retrofit it later.
- Write tests that mimic real user behavior, not just unit tests that call functions in isolation. If a user submits a form, test the full path from input to response.
- Move every configuration value that might change between environments into a config file or environment variable. No hardcoded URLs, timeouts, or credentials.

### Before deployment

- Write down the deployment steps as you perform them. If something goes wrong mid-deployment, you need to know exactly what state the system is in.
- Create a simple rollback plan. Know which backup you will restore, how you will restart the service, and how you will verify that the rollback succeeded. Test it in staging before you need it in production.

### After deployment

- Monitor your logs for one to two hours after deployment. Look for error rates, unexpected warnings, and any behavior that was not present before the release. New noise in the logs after a deployment is a signal worth investigating.
- Write a brief retrospective. What went as expected? What surprised you? What would you do differently next time? A short document written while the details are fresh is worth more than a thorough one written a week later.
