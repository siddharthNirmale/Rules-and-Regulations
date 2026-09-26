# Software Architecture & Structural Rules

> **Scope**: High cohesion, loose coupling, separation of concerns, YAGNI, and dependency directionality.

---

## Rule ARCH-01: The Rule of Three (Avoid Premature Abstraction) [MUST]
- **Rule**: Developers and AI agents MUST NOT create generic abstraction layers, plugin architectures, or dynamic factory registries for single-use operations. Write concrete code first; introduce an abstraction ONLY when three distinct, concrete implementations demonstrate identical duplicated logic.
- **Reason**: AI agents frequently generate abstract base classes, generic interfaces, and adapter factories in an attempt to look "architectural." Premature abstractions are almost always wrong because they guess future requirements incorrectly, locking the codebase into clumsy indirections that are painful to delete.
- **Risk**: Massive cognitive complexity, slow development velocity, and opaque call chains.
- **Correct Approach**:
  ```typescript
  // ❌ BAD: Over-engineered abstract factory for a single Stripe payment
  interface IPaymentStrategy<TInput, TResult> {
    execute(input: TInput): Promise<TResult>;
  }
  class PaymentStrategyFactory {
    static getStrategy(type: 'STRIPE'): IPaymentStrategy<StripeCharge, ChargeResult> { ... }
  }

  // ✅ CORRECT: Concrete, direct, easily understandable service
  export class PaymentService {
    constructor(private readonly stripeClient: Stripe) {}

    async processCharge(amountCents: number, customerId: string): Promise<Stripe.Charge> {
      return this.stripeClient.charges.create({ amount: amountCents, customer: customerId });
    }
  }
  ```
- **Exceptions**: Core framework architecture designed explicitly as a public SDK or plugin system for external developers.

---

## Rule ARCH-02: Unidirectional Dependency Flow (Clean Architecture) [MUST]
- **Rule**: Dependencies between modules MUST flow inward toward the domain core. High-level business policies MUST NOT import low-level infrastructure details (database drivers, HTTP framework objects, AWS SDK instances).
- **Reason**: When business logic imports database clients or Express request objects, you cannot test business logic without mocking the database driver, and you cannot swap protocols or upgrade dependencies without rewriting business rules.
- **Risk**: Brittle codebases, impossible unit testing, and framework lock-in.
- **Correct Approach**:
  ```
  Transport (Controllers / CLI) ➔ Application Services ➔ Domain Entities
                                          ⬇
                                  Repository Interfaces
                                          ⬆
                              Infrastructure (Postgres/Redis)
  ```
- **Exceptions**: Small scripts, prototypes, or single-file micro-utilities.

---

## Rule ARCH-03: Zero Circular Dependencies [MUST]
- **Rule**: Two modules MUST NOT import each other, either directly (`A -> B -> A`) or transitively (`A -> B -> C -> A`).
- **Reason**: Circular imports cause unpredictable runtime initialization bugs where imported symbols resolve to `undefined`, break tree-shaking, and create memory leaks.
- **Risk**: Silent runtime crashes (`TypeError: Cannot read properties of undefined`) that pass static linting.
- **Correct Approach**:
  - Extract shared types or functions into an independent leaf module `C`.
  - Both `A` and `B` import from `C`.
  - Enforce automated detection via tools like `madge --circular src/`.
- **Exceptions**: None.

---

## Rule ARCH-04: Strict Module Boundaries & Public API Exports [SHOULD]
- **Rule**: Modules and feature directories SHOULD expose their public contract through a single entry point (e.g., `index.ts` / barrel export) and keep internal helper functions, private classes, and subcomponents un-exported.
- **Reason**: When external modules reach deep into another module's internal directory (`import { helper } from '@/features/auth/internal/utils/salt'`), internal refactoring becomes impossible without breaking unrelated files across the codebase.
- **Risk**: Sprawling coupling that prevents safe local refactoring.
- **Exceptions**: Micro-benchmarking or test files located within the same directory.
