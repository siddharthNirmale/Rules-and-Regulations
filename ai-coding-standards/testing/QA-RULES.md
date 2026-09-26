# Quality Assurance (QA) & Code Review Rubric

> **Authority**: Evaluation Standard for Code Reviews, PR Quality Gates, and AI Agent Self-Audits.

---

## 1. Severity Classification Matrix

Reviewers and automated bots MUST classify all defects using this standardized severity ladder:

| Severity Level | Definition | Impact | Merge Action |
| :--- | :--- | :--- | :--- |
| **S1 Critical** 🔴 | Vulnerability (SQLi, IDOR, RCE, secret exposure), data corruption, auth bypass | High security threat or immediate data loss | **Block Immediately**. Non-negotiable fix required. |
| **S2 High** 🟠 | Functional breakage, broken accessibility, unhandled exception in core workflow | Severe user disruption or business logic failure | **Block Merge**. Must be resolved prior to release. |
| **S3 Normal** 🟡 | Edge-case unhandled, missing index, performance regression, inconsistent error shape | Minor defect or degraded performance | **Must Resolve**. Requires explicit approval to defer. |
| **S4 Minor** 🔵 | Minor naming confusion, code duplication under 5 lines, formatting inconsistency | Code cleanliness and readability | Fix if fast; track as debt if non-trivial. |
| **S5 Nit / Info** ⚪ | Pure aesthetic preference, equivalent alternative implementation style | Subjective stylistic opinion | **Do Not Block**. Discretionary for author. |

> **Mandate**: An AI agent or human reviewer **MUST NOT** escalate an S5 stylistic opinion to an S1/S2 blocking issue. This prevents alert fatigue and preserves team velocity.

---

## Rule QA-01: Every Bug Fix MUST Include a Regression Test
- **Rule**: When fixing a reported bug, the developer or AI agent MUST first write a failing test that reproduces the bug, and then apply the fix to make the test pass. The regression test MUST be committed in the same pull request.
- **Reason**: Any bug that occurred in production can easily recur months later during refactoring unless an automated test permanently guards that behavior.
- **Risk**: Re-introducing solved production bugs during future releases.
- **Correct Approach**:
  - Step 1: Add test `it('correctly handles leap year February 29 dates')` -> Test FAILS (reproduces bug).
  - Step 2: Fix date parsing logic -> Test PASSES.
  - Step 3: Commit fix and test together.
- **Exceptions**: Bugs caused strictly by external infrastructure failure or cloud provider outages that cannot be simulated locally.

---

## Rule QA-02: Zero Tautological / Hollow Tests
- **Rule**: Tests MUST assert meaningful business outcomes, not merely confirm that mocked functions were invoked. A test that passes regardless of whether the business logic is correct or broken is a "Tautological Test" and is strictly prohibited.
- **Reason**: Tautological tests inflate code coverage metrics to 95%+ while providing zero actual bug-catching capability.
- **Risk**: False sense of security and undetected production breakages.
- **Correct Approach**:
  ```typescript
  // ❌ BAD: Tautological test (only tests that the mock was wired)
  it('processes payment', async () => {
    const mock = jest.fn().mockReturnValue(true);
    const service = new PaymentService(mock);
    await service.process();
    expect(mock).toHaveBeenCalled(); // Passes even if calculation was wrong!
  });

  // ✅ CORRECT: Asserts actual business calculation and state mutation
  it('applies statutory tax and deducts coupon from total charge', async () => {
    const service = new PaymentService(realOrFakedCalculator);
    const order = await service.calculateOrderTotal({ subtotal: 100, couponCode: 'SAVE20' });
    expect(order.finalCharge).toBe(88.00); // (100 - 20) + 10% tax = 88.00
  });
  ```
- **Exceptions**: Pure bridge/adapter classes whose sole responsibility is mapping arguments to third-party SDK calls.

---

## Rule QA-03: Mutation Resistance Verification
- **Rule**: To verify test quality, an engineer or AI agent SHOULD ask: *"If I invert this `>` to a `<`, or change this `+` to a `-`, will at least one test fail?"* If the tests still pass when the business logic is intentionally corrupted, the test suite is inadequate.
- **Reason**: High line coverage does not equal high assertion coverage. Mutation testing ensures that assertions actually validate data transformations.
- **Risk**: Untested logic branches hidden behind high test coverage metrics.
- **Correct Approach**: Run mutation testing tools (e.g. Stryker, Mutmut) in CI pipelines on core business packages.
- **Exceptions**: Non-critical UI presentation templates.
