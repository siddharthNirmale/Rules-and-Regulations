# Automated Testing & Test Design Rules

> **Scope**: The Testing Pyramid, unit/integration/E2E balance, deterministic fixtures, boundary testing, and mock discipline.

---

## The Testing Pyramid
Automated testing suites MUST balance test types according to cost, speed, and confidence:

```
      /\
     /  \    10% End-to-End Tests (Critical customer journeys)
    /────\
   /      \   20% Integration Tests (Database queries, API contracts, boundaries)
  /────────\
 /          \  70% Unit Tests (Domain logic, mathematical edge cases, state transitions)
/────────────\
```

---

## Rule TST-01: Test Observable Behavior, Not Implementation Details
- **Rule**: Tests MUST assert against public inputs and outputs (the observable contract of a function, component, or API). Tests MUST NOT assert on private member variables, internal method invocation counts, or ephemeral local variables.
- **Reason**: Tests tightly coupled to internal implementation details break every time the code is refactored, even when the external behavior remains 100% correct.
- **Risk**: Brittle test suites that create false alarm fatigue and discourage necessary refactoring.
- **Correct Approach**:
  ```typescript
  // ❌ BAD: Coupling test to internal private method call
  it('charges customer', async () => {
    const service = new PaymentService();
    const spy = jest.spyOn(service as any, '_validateCardFormat');
    await service.charge(customer, 100);
    expect(spy).toHaveBeenCalled(); // Brittle! Breaks if private method is inlined
  });

  // ✅ CORRECT: Asserting observable behavior and state
  it('charges customer successfully and records transaction', async () => {
    const service = new PaymentService(mockPaymentGateway, orderRepository);
    const result = await service.charge({ customerId: 'cust_1', amountCents: 10000 });

    expect(result.status).toBe('SUCCESS');
    expect(result.receiptUrl).toMatch(/^https:\/\/receipts\./);
  });
  ```
- **Exceptions**: White-box coverage analysis specifically targeting complex cryptographic or state machine internals.

---

## Rule TST-02: Strict Test Determinism (Zero Flakiness)
- **Rule**: Tests MUST execute deterministically, producing the identical result on every run regardless of the day, time zone, OS platform, or execution speed. Tests MUST NOT rely on live external network connections, unseeded pseudo-random numbers, or real system clocks.
- **Reason**: Flaky tests (tests that pass 98% of the time and fail randomly 2% of the time) erode engineering trust in the CI pipeline, leading teams to ignore test failures.
- **Risk**: Genuine production bugs merging unnoticed because developers assume a failed test is "just flaky."
- **Correct Approach**:
  - Mock external HTTP requests at the network level (using MSW - Mock Service Worker or Nock).
  - Freeze system time using test timers (`jest.useFakeTimers().setSystemTime(new Date('2024-01-01'))`).
  - Seed random number generators if testing probabilistic logic.
- **Exceptions**: Dedicated synthetic canary healthchecks running in production staging.

---

## Rule TST-03: Mandatory Boundary & Negative Edge Case Coverage
- **Rule**: Unit tests MUST NOT test only the happy path. Every non-trivial function MUST have test cases asserting behavior against:
  1. `null`, `undefined`, and empty inputs (`""`, `[]`, `{}`).
  2. Numerical boundary values (`0`, negative numbers, maximum safe integers).
  3. Format violations (invalid email, malformed UUID, extra payload keys).
  4. Network failure states (timeout, 500 status, connection refused).
- **Reason**: Almost all production outages and security vulnerabilities stem from unhandled edge cases, not happy path failures.
- **Risk**: Runtime exceptions and unhandled crashes under abnormal conditions.
- **Correct Approach**:
  ```typescript
  describe('calculateShippingFee', () => {
    // Happy path:
    it('calculates standard fee for 5kg domestic package', () => {
      expect(calculateShippingFee({ weightKg: 5, isDomestic: true })).toBe(15.00);
    });

    // Edge cases:
    it('throws InvalidWeightError for 0kg or negative weight', () => {
      expect(() => calculateShippingFee({ weightKg: 0, isDomestic: true })).toThrow(InvalidWeightError);
      expect(() => calculateShippingFee({ weightKg: -2, isDomestic: true })).toThrow(InvalidWeightError);
    });

    it('handles maximum allowable freight weight boundary (1000kg)', () => {
      expect(calculateShippingFee({ weightKg: 1000, isDomestic: true })).toBe(500.00);
      expect(() => calculateShippingFee({ weightKg: 1000.1, isDomestic: true })).toThrow(FreightLimitExceededError);
    });
  });
  ```
- **Exceptions**: None.

---

## Rule TST-04: Follow the AAA (Arrange-Act-Assert) Pattern
- **Rule**: Test bodies MUST be structured in three clear, distinct blocks:
  1. **Arrange**: Set up fixtures, inputs, and mocks.
  2. **Act**: Invoke the single method or endpoint under test.
  3. **Assert**: Verify the expected outcome and invariants.
- **Reason**: Consistent test structure makes tests instantly readable and isolates which step failed during test execution.
- **Risk**: Muddled test cases with interwoven actions and assertions that are hard to diagnose when they fail.
- **Correct Approach**:
  ```typescript
  it('updates customer phone number', async () => {
    // 1. Arrange
    const customer = await createTestCustomer();
    const newPhoneNumber = '+15551234567';

    // 2. Act
    const updated = await customerService.updatePhone(customer.id, newPhoneNumber);

    // 3. Assert
    expect(updated.phoneNumber).toBe(newPhoneNumber);
  });
  ```
- **Exceptions**: Complex integration workflows explicitly testing multi-step state machine transitions.
