# Design System & Component Architecture Rules

> **Standard**: Semantic Design Tokens, Atomic Hierarchy, Consistent Prop Interfaces, and Theme Adaptability.

---

## Rule DS-01: Exclusively Use Semantic Design Tokens
- **Rule**: Component styles MUST reference semantic design tokens (`var(--color-surface)`, `var(--color-primary)`, `text-foreground-muted`) rather than hardcoded hex colors (`#1e293b`), raw RGB values, or ad-hoc pixel numbers.
- **Reason**: Hardcoding hex values across dozens of components breaks dark-mode switching, theme customization, and prevents brand-wide design updates.
- **Risk**: Inconsistent styling, high refactoring costs when branding changes, and broken dark-mode color inversions.
- **Correct Approach**:
  ```css
  /* ❌ BAD: Hardcoded hex codes */
  .card {
    background: #ffffff;
    border: 1px solid #e2e8f0;
    color: #1e293b;
  }

  /* ✅ CORRECT: Semantic CSS custom property tokens */
  :root {
    --color-bg-surface: #ffffff;
    --color-border-subtle: #e2e8f0;
    --color-text-primary: #0f172a;
  }
  [data-theme='dark'] {
    --color-bg-surface: #1e293b;
    --color-border-subtle: #334155;
    --color-text-primary: #f8fafc;
  }

  .card {
    background: var(--color-bg-surface);
    border: 1px solid var(--color-border-subtle);
    color: var(--color-text-primary);
  }
  ```
- **Exceptions**: Specific brand logo SVGs with protected trademark guidelines.

---

## Rule DS-02: Consistent Component Prop Contracts
- **Rule**: Shared design system components MUST follow consistent, standardized prop names across the entire library:
  - `variant`: `'primary' | 'secondary' | 'outline' | 'ghost' | 'danger'`
  - `size`: `'sm' | 'md' | 'lg'`
  - `isLoading`: `boolean` (displays spinner, disables interaction)
  - `disabled`: `boolean` (lowers opacity, prevents events)
- **Reason**: Inconsistent prop names (`variant` on buttons vs `type` on badges vs `theme` on alerts) create severe cognitive friction for developers and AI agents alike.
- **Risk**: Frequent developer errors and broken component renderings.
- **Correct Approach**:
  ```tsx
  // ✅ CORRECT: Standardized props interface
  interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
    variant?: 'primary' | 'secondary' | 'outline' | 'danger';
    size?: 'sm' | 'md' | 'lg';
    isLoading?: boolean;
    leftIcon?: React.ReactNode;
    rightIcon?: React.ReactNode;
  }
  ```
- **Exceptions**: Unique domain-specific widgets with specialized parameters.

---

## Rule DS-03: Prefer Component Composition Over Prop Explosion
- **Rule**: Components SHOULD use composition (slots, `children`, compound subcomponents like `Dialog.Header`, `Dialog.Body`, `Dialog.Footer`) rather than accepting 25 optional boolean props to customize layout.
- **Reason**: "God components" with dozens of configuration props become brittle, difficult to type, and impossible to maintain. Compound components allow consumers to arrange layouts naturally.
- **Risk**: Exponential complexity in component render logic and frequent edge-case visual regressions.
- **Correct Approach**:
  ```tsx
  // ❌ BAD: Prop explosion
  <Card
    title="Profile"
    subtitle="User details"
    showEditButton={true}
    onEdit={() => {}}
    hasBorder={true}
    footerText="Updated 5m ago"
    headerAction={<Icon />}
  />

  // ✅ CORRECT: Compound component composition
  <Card>
    <Card.Header>
      <div>
        <Card.Title>Profile</Card.Title>
        <Card.Subtitle>User details</Card.Subtitle>
      </div>
      <Button variant="ghost" onClick={handleEdit}>Edit</Button>
    </Card.Header>
    <Card.Body>
      <UserProfileDetails />
    </Card.Body>
    <Card.Footer>
      <span className="text-sm text-slate-500">Updated 5m ago</span>
    </Card.Footer>
  </Card>
  ```
- **Exceptions**: Simple micro-atoms like `<Badge>` or `<Avatar>`.
