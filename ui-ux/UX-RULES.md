# User Experience (UX) & Interaction Rules

> **Scope**: User feedback loops, affordance, error recovery, empty states, and interaction ergonomics.

---

## Rule UX-01: Immediate Feedback on All User Actions (< 100ms) [MUST]
- **Rule**: Every user interaction (button press, form submit, toggle switch, tab change) MUST provide visual acknowledgment within **100 milliseconds**. For asynchronous tasks taking over 500ms, a spinner, progress bar, or skeleton loader MUST be displayed.
- **Reason**: The human brain perceives delays under 100ms as instantaneous. Delays exceeding 100ms without feedback make users question whether their click was registered, causing them to click repeatedly.
- **Risk**: Frustrated users and double-submitted form entries.
- **Correct Approach**:
  - Immediate `:active` button scale down (`transform: scale(0.98)`).
  - Submit button disables and displays an in-line spinner during network requests.
  - Optimistic UI updates where safe (e.g. toggling a "Like" heart or bookmark icon immediately).
- **Exceptions**: None.

---

## Rule UX-02: Actionable Error Messages with Clear Recovery Paths [MUST]
- **Rule**: Validation and operational error messages MUST explain: (1) what happened; (2) why it happened; and (3) what specific action the user can take to fix or recover from it. Generic error messages like `"An error occurred"`, `"Submission failed"`, or `"Invalid input"` are strictly prohibited on user forms.
- **Canonical Boundary Reference**: See [ERROR-HANDLING-RULES.md](../backend/ERROR-HANDLING-RULES.md#rule-err-02-sanitize-system-errors-provide-actionable-validation-details-must).
- **Reason**: Users cannot self-correct without knowing which field or requirement was violated.
- **Risk**: User abandonment and increased customer support tickets.
- **Correct Approach**:
  ```tsx
  // ❌ BAD: Vague, unhelpful
  <Alert message="Payment failed." />

  // ✅ CORRECT: Specific, explains root cause, provides recovery action
  <Alert
    variant="error"
    title="Payment Card Declined"
    description="Your bank declined the transaction due to insufficient funds. Please select an alternate payment method or contact your card issuer."
    actionLabel="Try Another Card"
    onAction={() => openPaymentModal()}
  />
  ```
- **Security Precedence Exception**: On authentication forms (login / password reset), error messages MUST NOT reveal whether the email address exists in the system (`"Invalid email or password"`), in order to prevent username enumeration.

---

## Rule UX-03: Meaningful Empty States with Direct Calls to Action [MUST]
- **Rule**: When a table, feed, or collection contains zero records, the application MUST render an informative empty state containing: (1) a descriptive graphic or icon; (2) a title explaining that the collection is empty; (3) explanation of how items appear; and (4) a primary action button to create the first item.
- **Reason**: A blank white screen or raw empty table header looks broken or frozen to users.
- **Risk**: Confused onboarding users who do not know what step to take next.
- **Correct Approach**:
  ```tsx
  // ✅ CORRECT
  function ProjectsEmptyState() {
    return (
      <div className="flex flex-col items-center justify-center p-12 text-center border-2 border-dashed border-slate-300 rounded-xl">
        <FolderPlusIcon className="w-12 h-12 text-slate-400 mb-3" />
        <h3 className="text-lg font-semibold text-slate-800">No projects created yet</h3>
        <p className="text-sm text-slate-500 max-w-sm mt-1 mb-4">
          Get started by creating your first workspace to collaborate with your team in real time.
        </p>
        <button type="button" onClick={openNewProjectModal} className="btn-primary">
          Create New Project
        </button>
      </div>
    );
  }
  ```
- **Exceptions**: Search filter results where no matches are found (which should display: *"No results matching 'keyword' — Clear filters"*).

---

## Rule UX-04: Destructive Safeguards (Confirmations & Undo) [MUST]
- **Rule**: Permanent or destructive operations (deleting a workspace, resetting an account, terminating an active subscription) MUST require explicit user confirmation via a modal dialog or provide a timed "Undo" grace period (toast banner).
- **Reason**: Accidental clicks, trackpad slips, or misunderstandings can result in irreversible data loss.
- **Risk**: Permanent loss of user data and catastrophic customer dissatisfaction.
- **Correct Approach**:
  - For high-stakes deletions (e.g. deleting an entire production cluster), require typing the resource name into an input to enable the delete button.
  - For routine item deletions (e.g. removing an email or tag), offer a 5-second "Undo" snackbar.
- **Exceptions**: Reversible actions with instant local storage recovery.
