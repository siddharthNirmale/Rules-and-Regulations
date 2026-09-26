# Continuous Integration & Delivery (CI/CD) Rules

> **Scope**: Automated build pipelines, PR verification gates, artifact immutability, and deployment safety.

---

## The Mandatory CI Pipeline Stages
Every Pull Request and commit to the default branch MUST execute and pass through five sequential automated quality gates before merge:

```
[ Stage 1: Static Analysis ] ➔ [ Stage 2: Security Audit ] ➔ [ Stage 3: Test Suite ] ➔ [ Stage 4: Build ] ➔ [ Stage 5: Artifact ]
- ESLint / Ruff / Prettier    - Dependency CVE scan         - Unit & Integration      - Compiler checks  - Tagged image
- Typecheck (tsc --noEmit)    - Secret scanner (gitleaks)   - 100% deterministic pass  - Assets bundling  - Signed SHA digest
```

---

## Rule CI-01: Branch Protection & Zero Direct Pushes to Main
- **Rule**: Direct commits or force pushes to the primary branch (`main`, `master`, `production`) MUST be blocked at the repository settings level. All changes MUST arrive via Pull Requests that satisfy mandatory automated CI checks and code review approvals.
- **Reason**: Allowing direct pushes bypasses automated linting, security scans, and test suites, creating unverified production regressions.
- **Risk**: Immediate un-reviewed production outages and broken main branches.
- **Correct Approach**: Configure GitHub/GitLab Branch Protection:
  - Require status checks to pass before merging (`typecheck`, `lint`, `test`, `audit`).
  - Require linear commit history or squash merging.
  - Require at least one peer code review approval.
- **Exceptions**: None.

---

## Rule CI-02: Immutable Artifact Tagging (Never Deploy `latest`)
- **Rule**: Production deployment artifacts (Docker container images, serverless zip packages, compiled binaries) MUST be tagged with immutable identifiers (e.g. the full Git commit SHA: `app:sha-a8f3b21` or semantic version `app:v2.4.1`). Images MUST NOT be deployed using the mutable `:latest` tag.
- **Reason**: The `:latest` tag is mutable; it changes whenever another image is pushed. If a pod restarts in a cluster, it may pull a different version of `:latest`, resulting in pods within the same cluster running different versions of the code.
- **Risk**: Non-reproducible deployments, broken rolling rollbacks, and inability to trace which code is running.
- **Correct Approach**:
  ```yaml
  # GitHub Actions step tagging image with Git SHA
  - name: Build and Push Docker Image
    run: |
      docker build -t registry.example.com/app:sha-${{ github.sha }} .
      docker push registry.example.com/app:sha-${{ github.sha }}
  ```
- **Exceptions**: Local developer testing environments.

---

## Rule CI-03: Fast Feedback Loops (< 10 Minutes Total CI Time)
- **Rule**: Standard CI pull request pipelines MUST complete execution within **10 minutes**. Use dependency caching (caching `~/.npm`, `~/.cargo`, `~/.cache/pip`), parallel job matrices, and test sharding to maintain rapid feedback.
- **Reason**: When CI pipelines take 30+ minutes, developers context-switch, review cycles grind to a halt, and team velocity collapses.
- **Risk**: Blocked engineering teams and temptation to skip or bypass CI checks.
- **Correct Approach**:
  ```yaml
  # Use official cache actions to preserve dependency stores
  - uses: actions/setup-node@v4
    with:
      node-version: 20
      cache: 'npm' # Caches ~/.npm automatically
  ```
- **Exceptions**: Extended nightly end-to-end load tests and comprehensive fuzzing suites.
