# Dependency & Supply-Chain Security Rules

> **Scope**: Third-party package evaluation, lockfile integrity, automated CVE scanning, and supply-chain attack prevention.

---

## The Supply Chain Threat
Modern web applications consist of up to 90% open-source code pulled from npm, PyPI, Crates.io, and Maven Central. An attacker compromising a single transitive dependency can execute arbitrary code inside build pipelines, steal production secrets, or backdoor deployed applications.

---

## Rule DEP-01: Mandatory Lockfile Commitment & Deterministic Installs [MUST]
- **Rule**: Lockfiles (`package-lock.json`, `pnpm-lock.yaml`, `poetry.lock`, `Cargo.lock`, `go.sum`) MUST be committed to version control. Production builds and CI pipelines MUST use frozen/deterministic install commands (`npm ci`, `pnpm install --frozen-lockfile`, `cargo build --locked`).
- **Reason**: Regular `npm install` can pull newer minor or patch versions that contain unvetted regressions or malicious backdoored updates.
- **Risk**: Build non-determinism and silent injection of malicious package versions.
- **Correct Approach**:
  ```bash
  # ❌ BAD: May pull unvetted new patch versions
  npm install

  # ✅ CORRECT: Deterministic build matching exact lockfile hashes
  npm ci
  ```
- **Exceptions**: None.

---

## Rule DEP-02: Zero Dependencies for Trivial Native Operations [MUST]
- **Rule**: Developers and AI agents MUST NOT install external packages for operations natively provided by standard language libraries and modern runtimes.
- **Reason**: Trivial packages (`left-pad`, `is-even`, `uuid`, `chalk` for simple colors) expand the attack surface, increase install times, and create long-term maintenance burdens.
- **Risk**: Supply chain compromise of micro-packages (e.g. the `event-stream` incident).
- **Prohibited Dependencies**:
  - `left-pad`, `pad-left` -> Use `String.prototype.padStart()`
  - `is-even`, `is-odd` -> Use `% 2 === 0`
  - `uuid` -> Use native `crypto.randomUUID()`
  - `querystring` -> Use native `URLSearchParams`
  - `axios` -> Use native `fetch()` unless interceptors/caching are proven necessary
- **Exceptions**: Complex domain algorithms (e.g. `libphonenumber-js`, `bignumber.js`).

---

## Rule DEP-03: Mandatory Automated Vulnerability Scanning in CI [MUST]
- **Rule**: Continuous Integration pipelines MUST execute an automated vulnerability audit (e.g. `npm audit --audit-level=high`, `pip-audit`, `snyk test`, `cargo audit`) on every pull request. Builds MUST fail if unresolved High or Critical CVEs are detected.
- **Reason**: Automated scanning identifies known vulnerabilities in dependencies before they reach production.
- **Risk**: Deploying known, weaponized vulnerabilities that allow remote exploit scripts to compromise servers.
- **Correct Approach**:
  ```yaml
  # GitHub Actions CI Step
  - name: Audit Dependencies
    run: npm audit --audit-level=high --production
  ```
- **Exceptions**: Documented false positives where a CVE affects an un-imported or unreachable code path, accompanied by an explicit override comment in `.auditrc` or `package.json`.

---

## Rule DEP-04: Strict Package Vetting Criteria [SHOULD]
- **Rule**: Before adding any new third-party dependency, the developer or AI agent SHOULD verify:
  1. **Weekly Downloads / Popularity**: At least 50,000+ weekly downloads (for npm).
  2. **Active Maintenance**: Commits or releases within the past 12 months.
  3. **Permissive License**: MIT, Apache 2.0, BSD, or ISC. Copyleft licenses (GPL, AGPL) MUST NOT be introduced into proprietary commercial software without legal approval.
  4. **Exact Name Verification**: Check for typosquatting (e.g. `lodsh` instead of `lodash`).
- **Reason**: Attackers publish malicious typosquatted packages that execute post-install scripts to exfiltrate `.env` files.
- **Risk**: Intellectual property contamination from GPL licenses and credential theft from malicious packages.
- **Exceptions**: Official internal corporate packages published to private enterprise registries.
