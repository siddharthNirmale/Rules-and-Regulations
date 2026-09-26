# Accessibility (a11y) Rules — WCAG 2.2 AA Standards

> **Authority**: Mandatory Compliance  
> **Standard**: W3C Web Content Accessibility Guidelines (WCAG) 2.2 Level AA

---

## Rule A11Y-01: Explicit Labeling of All Interactive Form Controls
- **Rule**: Every `<input>`, `<select>`, and `<textarea>` MUST have an associated `<label>` element linked via `for`/`htmlFor` and `id`, or an explicit `aria-label`/`aria-labelledby` attribute.
- **Reason**: Assistive technologies (screen readers) announce the label when an input receives focus. Placeholder attributes disappear when text is typed and are not reliably announced as accessible names.
- **Risk**: Blind and low-vision users cannot identify what information a form input expects.
- **Correct Approach**:
  ```tsx
  // ❌ BAD
  <input type="text" placeholder="First Name" />

  // ✅ CORRECT
  <label htmlFor="first-name" className="block text-sm font-medium">First Name</label>
  <input id="first-name" name="firstName" type="text" className="..." />

  // ✅ CORRECT (Visually hidden for icon-only triggers)
  <button type="button" aria-label="Close dialog">
    <CloseIcon aria-hidden="true" />
  </button>
  ```
- **Exceptions**: None. Every interactive control must have an accessible name.

---

## Rule A11Y-02: Never Remove the Focus Outline Without a High-Contrast Alternative
- **Rule**: Developers and agents MUST NOT apply `outline: none` or `outline: 0` unless an explicit, high-contrast `:focus-visible` outline or ring is simultaneously provided.
- **Reason**: Keyboard-only users rely entirely on the visual focus indicator to determine which link, button, or input currently has focus.
- **Risk**: Stripping focus indicators renders web pages unusable for motor-impaired and keyboard navigation users.
- **Correct Approach**:
  ```css
  /* ❌ BAD */
  button:focus { outline: none; }

  /* ✅ CORRECT */
  button:focus-visible {
    outline: 2px solid #2563eb;
    outline-offset: 2px;
  }
  ```
- **Exceptions**: None. Focus indicators are mandatory for keyboard accessibility.

---

## Rule A11Y-03: Minimum Color Contrast Ratios (WCAG AA)
- **Rule**: Normal text (under 18pt or 14pt bold) MUST have a contrast ratio of at least **4.5:1** against its background. Large text (>= 18pt or >= 14pt bold) and graphical user interface components/borders MUST have a ratio of at least **3:1**.
- **Reason**: Users with low vision, color blindness, or viewing screens in bright sunlight cannot read low-contrast text (e.g., light gray text on white backgrounds).
- **Risk**: Unreadable interfaces, poor usability, and legal non-compliance under the Americans with Disabilities Act (ADA) and European Accessibility Act (EAA).
- **Correct Approach**:
  ```css
  /* ❌ BAD: Contrast 2.1:1 on white background */
  color: #94a3b8; background-color: #ffffff;

  /* ✅ CORRECT: Contrast 7.0:1 on white background */
  color: #334155; background-color: #ffffff;
  ```
- **Exceptions**: Inactive/disabled interface components or purely decorative logos.

---

## Rule A11Y-04: Do Not Rely on Color Alone for Meaning
- **Rule**: Color MUST NOT be used as the only visual means of conveying information, indicating an action, prompting a response, or distinguishing a visual element.
- **Reason**: Color-blind users (affecting up to 8% of males) cannot distinguish between red and green status indicators if not accompanied by text, icons, or underline styling.
- **Risk**: Color-blind users cannot detect form validation errors or critical system warnings.
- **Correct Approach**:
  ```tsx
  // ❌ BAD: Only red border and color to indicate error
  <input className="border-red-500 text-red-500" />

  // ✅ CORRECT: Color + Error Icon + Accessible Error Text linked via aria-describedby
  <div>
    <input
      id="email"
      aria-invalid="true"
      aria-describedby="email-error"
      className="border-red-600 focus-visible:outline-red-600"
    />
    <p id="email-error" role="alert" className="mt-1 flex items-center text-sm text-red-700">
      <ExclamationIcon aria-hidden="true" className="mr-1 h-4 w-4" />
      Please enter a valid email address.
    </p>
  </div>
  ```
- **Exceptions**: None.

---

## Rule A11Y-05: Modals MUST Trap Focus and Restore Focus on Close
- **Rule**: When a modal or dialog opens: (1) focus MUST move to the first interactive element inside the modal; (2) focus MUST be trapped inside the modal during Tab navigation; (3) pressing `Escape` MUST close the modal; (4) focus MUST return to the trigger element that opened the modal upon closing.
- **Reason**: Without focus trapping, keyboard users tab through the invisible background page behind the modal overlay.
- **Risk**: Keyboard users become trapped or disoriented, unable to navigate back to the dialog or trigger.
- **Correct Approach**: Use native HTML5 `<dialog>` with `.showModal()`, or an accessible headless primitive (Radix UI, Headless UI, Ariakit).
  ```tsx
  // ✅ CORRECT (HTML5 native dialog)
  function Modal({ isOpen, onClose, children }) {
    const dialogRef = useRef<HTMLDialogElement>(null);

    useEffect(() => {
      const dialog = dialogRef.current;
      if (!dialog) return;
      if (isOpen) {
        dialog.showModal(); // Native focus trap and Escape handler!
      } else {
        dialog.close();
      }
    }, [isOpen]);

    return (
      <dialog ref={dialogRef} onClose={onClose} className="rounded-lg p-6 backdrop:bg-black/50">
        {children}
        <button type="button" onClick={onClose}>Close</button>
      </dialog>
    );
  }
  ```
- **Exceptions**: Non-modal popovers or modeless tooltips that do not restrict interaction with the background document.

---

## Rule A11Y-06: Respect User Motion Preferences (`prefers-reduced-motion`)
- **Rule**: All transitions, auto-playing media, and non-essential parallax or layout animations MUST respect the `prefers-reduced-motion` media query.
- **Reason**: Users with vestibular disorders, motion sickness, or epilepsy suffer nausea, dizziness, or seizures from intense animated motion.
- **Risk**: Severe physical discomfort or medical emergencies for users with vestibular system conditions.
- **Correct Approach**:
  ```css
  /* ✅ CORRECT */
  @media (prefers-reduced-motion: reduce) {
    *, ::before, ::after {
      animation-duration: 0.01ms !important;
      animation-iteration-count: 1 !important;
      transition-duration: 0.01ms !important;
      scroll-behavior: auto !important;
    }
  }
  ```
- **Exceptions**: Essential state indicator animations (e.g., an instant opacity change rather than a bouncing motion).
