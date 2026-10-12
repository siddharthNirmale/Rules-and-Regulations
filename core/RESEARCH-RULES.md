# Research Methodology & Source Quality Hierarchy

> **Status**: Mandatory Standard  
> **Applies to**: All AI Coding Agents, Autonomous Systems, Technical Architects, and Human Contributors  
> **Scope**: Evaluating, ranking, and citing external technical sources, researching best practices, resolving conflicting claims, and justifying codebase rules.

---

## 1. The Core Philosophy

When researching technical practices, adopting dependencies, drafting architecture guidelines, or validating platform APIs, software engineering decisions must be grounded in **verifiable, authoritative, and empirical evidence**. 

Neither human developers nor AI coding agents may rely on unsupported assertions, outdated tutorials, or popular consensus as proof of technical correctness. Every technical recommendation carries long-term operational and security liability; therefore, evidence strength must match the criticality of the recommendation.

---

## 2. The 5-Tier Source Quality Hierarchy

All external sources used to justify, design, or verify coding standards, security controls, architectural choices, and dependency additions MUST be evaluated and ranked according to the following strict hierarchy:

| Tier | Tier Classification | Qualifying Sources & Specifications | Operational Role & Evidentiary Weight |
| :---: | :--- | :--- | :--- |
| **Tier 1** | **Primary & Authoritative Sources** | • **Official Language Specs**: ECMAScript (TC39), Python PEPs, Rust RFCs, Go spec, ISO C++.<br>• **Official Framework & Runtime Docs**: React, Next.js, Node.js, PostgreSQL, Docker.<br>• **Recognized Security Standards**: OWASP (ASVS, Top 10, LLM Top 10), NIST (SP 800-218 SSDF, SP 800-53, SP 800-63), MITRE CWE.<br>• **Protocol & Networking Specs**: IETF RFCs (HTTP, TLS, OAuth, WebSockets), W3C Recommendations.<br>• **Primary Research**: Formal papers published by original creators or core project maintainers. | **Ground Truth (Maximum Weight)**<br>Mandatory baseline for security, architecture, performance bounds, and platform API contracts. Non-negotiable. |
| **Tier 2** | **Established Technical Literature** | • **Peer-Reviewed Software Engineering Research**: IEEE Transactions on Software Engineering (TSE), ACM TOSEM, ICSE, USENIX Security.<br>• **Foundational Engineering Literature**: Seminal works by recognized domain authorities (Martin Fowler, Robert C. Martin, Michael Feathers, Martin Kleppmann).<br>• **Maintainer Technical Publications**: Official engineering blogs by maintainer teams (Go Blog, Rust Blog, Chromium Dev, AWS Architecture Blog).<br>• **Empirical Software Studies**: Peer-reviewed empirical methods (Kitchenham Evidence-Based Software Engineering, DORA Accelerate). | **Deep Guidance (High Weight)**<br>Authoritative foundation for architectural trade-offs, refactoring strategies, failure modes, and empirical outcomes. |
| **Tier 3** | **High-Quality Repositories** | • **Active, High-Quality GitHub Repositories**: Repositories with rigorous test coverage, automated CI/CD, published RFC processes, and active security triaging.<br>• **Curated Engineering Standards**: Production-tested guidelines from established engineering teams (Google Engineering Practices, GitHub Engineering Standards, Node Best Practices).<br>• **Vetted Coding Agent Instruction Repositories**: Proven prompt/rule suites evaluated by maintenance activity, commit cadence, and real-world usefulness, **never by stars or forks alone**. | **Applied Patterns (Medium-High Weight)**<br>Practical implementations, battle-tested configuration patterns, linter rules, and real-world project scaffolding. |
| **Tier 4** | **Developer Communities** | • **Community Discussion Forums**: Stack Overflow, Reddit (`r/programming`, `r/ExperiencedDevs`), Hacker News, dev.to discussions.<br>• **Repository Issue Trackers & PRs**: Real-world triage discussions, bug reports, and code review debates.<br>• **Engineering Postmortems**: Public postmortems, incident retrospectives, and outage analyses from production teams. | **Discovery & Edge Cases (Context Only)**<br>Valuable for uncovering real-world friction, edge cases, recurring mistakes, and migration hurdles. **MUST be validated against Tier 1 or Tier 2 sources before adopting recommendations.** |
| **Tier 5** | **Low-Confidence Sources** | • **Unsupported Personal Opinions**: Subjective hot-takes, unsubstantiated claims on social media.<br>• **Outdated Tutorials & Articles**: Blog posts demonstrating deprecated syntax or obsolete major versions.<br>• **Promotional & Marketing Content**: Vendor whitepapers or SEO articles designed primarily to sell proprietary tooling.<br>• **Unverified AI-Generated Content**: Automated content farms, unreviewed LLM output, or hallucinated summaries lacking reproducible tests. | **Zero Ground Truth (Prohibited)**<br>**MUST NEVER** be relied upon as the sole justification for any architectural rule, security control, or production change. |

---

## 3. The 5 Evaluation Dimensions

Every research source MUST be appraised along five orthogonal dimensions before its claims are accepted:

```
┌──────────────┐     ┌──────────────┐     ┌────────────────────┐     ┌─────────────┐     ┌───────────────┐
│  Authority   │ ──► │ Reliability  │ ──► │ Technical Accuracy │ ──► │   Recency   │ ──► │   Relevance   │
└──────────────┘     └──────────────┘     └────────────────────┘     └─────────────┘     └───────────────┘
```

1. **Authority**:
   - Is the source the direct author, maintainer, or standardizing body for the technology?
   - What institutional or academic backing substantiates the author's expertise?
2. **Reliability**:
   - Does the publisher have an established track record of editorial review, peer review, or formal errata?
   - Is the source prone to unedited modifications or sponsored bias?
3. **Technical Accuracy**:
   - Are claims supported by reproducible code snippets, automated test cases, mathematical proofs, or empirical benchmarks?
   - Does the source acknowledge edge cases, constraints, and operational trade-offs?
4. **Recency & Version-Specificity**:
   - Does the documentation match the exact active major version targeted by the project?
   - For fast-moving ecosystems (e.g., JavaScript runtimes, cloud SDKs, AI toolchains), verify release notes; never apply patterns from superseded major versions.
5. **Relevance**:
   - Does the source address the specific scale, architecture, runtime environment, and threat model of the current codebase?
   - Avoid applying distributed microservice patterns to single-process monoliths or vice versa.

---

## 4. Normative Conformance & Claim Classification

Technical recommendations and repository guidelines MUST clearly distinguish between the following four requirement tiers:

| Requirement Classification | Conformance Level | Evidentiary Requirement | Typical Applications |
| :--- | :---: | :--- | :--- |
| **Mandatory Standards** | `[MUST]` / `[MUST NOT]` | Supported by Tier 1 standards or universal security invariants. | SQL injection defense, zero hardcoded secrets, input schema validation, CSRF defenses. |
| **Established Best Practices** | `[SHOULD]` / `[SHOULD NOT]` | Supported by Tier 1 or Tier 2 literature. Valid exceptions require documented technical rationale. | Layered architecture, cognitive complexity limits, 70/20/10 test pyramid, structured JSON logging. |
| **Context-Dependent Recommendations** | `[CONTEXT-DEPENDENT]` | Supported by Tier 1, 2, or 3 sources, but applicability depends on project scale, latency targets, or runtime. | Row-Level Security vs. application multi-tenancy, server vs. client state caching, microservices vs. monolith. |
| **Subjective Preferences** | `[MAY]` / `[OPTIONAL]` | Syntactic or stylistic conventions where multiple valid patterns exist without affecting correctness. | Code formatting style (spaces vs. tabs), file naming conventions, import ordering. |

---

## 5. Source Conflict Resolution Protocol

When authoritative sources disagree, contributors and AI agents **MUST NOT** blindly choose based on tier ranking alone. Instead, execute the following conflict resolution procedure:

1. **Investigate Version Differences**:
   - Verify whether one source addresses an older major version (e.g., React 17 vs. 19, Node 16 CommonJS vs. Node 22 ESM, Python 2 vs. Python 3).
   - Prefer the version-specific documentation matching the project's explicit runtime target.
2. **Examine Environmental Assumptions & Scale**:
   - Identify whether Source A assumes high-throughput distributed microservices while Source B assumes single-node SQLite persistence.
   - Align recommendations with the actual constraints and infrastructure of the repository.
3. **Evaluate Threat Models & Safety Goals**:
   - When a performance recommendation conflicts with a security standard, invoke the **Priority & Conflict Resolution Hierarchy** from `MASTER-RULES.md`:
     ```text
     1. Safety and Security
     2. Functional Correctness
     ...
     8. Measured Performance & Efficiency
     ```
   - Security and correctness strictly outrank performance and developer convenience.
4. **Document the Trade-Off**:
   - When a non-obvious conflict is resolved, document the rationale in an Architecture Decision Record (ADR) citing both sources and explaining why one was chosen.

---

## 6. Citation Integrity & Anti-Fabrication Standards [MUST]

To maintain technical credibility and prevent hallucination drift, all human contributors and AI agents MUST abide by the following citation invariants:

- **Zero Hallucinated Citations**: Never invent author names, URLs, RFC numbers, book titles, or adoption metrics. Every citation MUST be verifiable.
- **Trace to the Primary Reference**: When citing a concept (e.g., Twelve-Factor App, Clean Architecture, OWASP Top 10), link directly to the primary standard or author's publication, rather than secondary blogs that merely summarize it.
- **Flag Inaccessible or Unverifiable Sources**: If a reference cannot be accessed or verified (due to paywalls, offline operation, or broken links), explicitly flag it:
  ```markdown
  <!-- [UNVERIFIED - INACCESSIBLE]: Source XYZ could not be accessed directly; claims remain unconfirmed. -->
  ```
- **Prohibition of Popularity Fallacy**: A high star count on GitHub or upvote count on Reddit does not prove technical correctness, security, or maintainability.

---

## 7. Required Research Output Schema

When proposing any major rule, architectural standard, or repository improvement, the proposal MUST follow this structured schema:

```markdown
### [Proposed Rule / Improvement Title]
- **Proposed Improvement**: Detailed description of the rule and the exact problem it solves.
- **Priority**: Critical | High | Medium | Low.
- **Supporting Source Links**: 
  - [Source 1 Title](URL): Specific claim substantiated.
  - [Source 2 Title](URL): Specific claim substantiated.
- **Evidence Strength**: High | Medium | Low.
- **Source Tier & Caveats**:
  - Source Tier: Tier 1 | Tier 2 | Tier 3 | Tier 4.
  - Limitations, disagreements, version dependencies, or context-dependent trade-offs.
```
