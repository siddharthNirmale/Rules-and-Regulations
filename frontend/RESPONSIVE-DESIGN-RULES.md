# Responsive Design & Multi-Device Rules

> **Standard**: Mobile-First, Fluid Layouts, Touch Accessibility, and Breakpoint Consistency.

---

## Standard Breakpoint System
Applications MUST adhere to consistent, standard min-width breakpoints:
- **Mobile (Base)**: `0px` to `639px` (Default styles)
- **Tablet (`sm`/`md`)**: `640px` / `768px`
- **Desktop (`lg`)**: `1024px`
- **Wide (`xl`/`2xl`)**: `1280px` / `1536px`

---

## Rule RESP-01: Mobile-First CSS Architecture
- **Rule**: Styles MUST be written mobile-first using `min-width` media queries rather than desktop-down `max-width` overrides.
- **Reason**: Mobile-first styling provides cleaner inheritance, fewer CSS rule overrides, and faster performance on low-power mobile devices since base styles apply immediately without parsing complex nested queries.
- **Risk**: Fragile desktop-down stylesheets with cascading specificity bugs and excessive `!important` declarations.
- **Correct Approach**:
  ```css
  /* ❌ BAD: Desktop-down max-width */
  .grid { display: grid; grid-template-columns: repeat(4, 1fr); }
  @media (max-width: 768px) {
    .grid { grid-template-columns: 1fr; }
  }

  /* ✅ CORRECT: Mobile-first min-width */
  .grid { display: grid; grid-template-columns: 1fr; gap: 1rem; }
  @media (min-width: 768px) {
    .grid { grid-template-columns: repeat(2, 1fr); }
  }
  @media (min-width: 1024px) {
    .grid { grid-template-columns: repeat(4, 1fr); }
  }
  ```
- **Exceptions**: Print stylesheets (`@media print`) or rare device-specific max-height orientations.

---

## Rule RESP-02: Zero Horizontal Overflow
- **Rule**: Layout containers MUST NOT have fixed pixel widths (`width: 1200px`) that cause horizontal scrolling on viewports narrower than the declared width.
- **Reason**: Unintended horizontal scrollbars create disorienting navigation, clip content, and disrupt mobile gestures.
- **Risk**: Immediate user bounce and poor mobile usability.
- **Correct Approach**:
  ```css
  /* ❌ BAD */
  .main-content { width: 1140px; }

  /* ✅ CORRECT */
  .main-content {
    width: 100%;
    max-width: 72rem; /* 1152px max width */
    margin-inline: auto;
    padding-inline: clamp(1rem, 4vw, 2rem);
  }
  ```
- **Anti-Pattern Warning**: NEVER mask horizontal overflow by slapping `overflow-x: hidden` onto `<body>` or `<html>` to hide misaligned elements. Fix the underlying layout width.

---

## Rule RESP-03: Minimum Touch Target Dimensions (44x44px)
- **Rule**: All interactive controls (buttons, links, form inputs, toggle switches) on mobile viewports MUST occupy a minimum clickable footprint of **44x44 CSS pixels**.
- **Reason**: Human fingertips average 10mm in diameter. Small touch targets cause accidental taps on adjacent links, frustrating users.
- **Risk**: High error rates on mobile e-commerce checkout and navigation forms.
- **Correct Approach**:
  ```css
  /* ✅ CORRECT: Accessible button footprint */
  .touch-target {
    min-width: 2.75rem; /* 44px */
    min-height: 2.75rem; /* 44px */
    display: inline-flex;
    align-items: center;
    justify-content: center;
  }
  ```
- **Exceptions**: In-line text links inside body paragraphs, where surrounding text spacing provides sufficient tap distinction.

---

## Rule RESP-04: Fluid Typography with CSS `clamp()`
- **Rule**: Headings and fluid display typography SHOULD scale smoothly between mobile and desktop viewports using CSS `clamp()` rather than jarring step changes at breakpoints.
- **Reason**: Smooth interpolation prevents text clipping, awkward word wraps, and excessive media query clutter.
- **Risk**: Headings that wrap to 5 lines on mobile or look tiny on 4K ultrawide monitors.
- **Correct Approach**:
  ```css
  /* ✅ CORRECT: Smoothly scales from 2rem (32px) on mobile to 3.75rem (60px) on desktop */
  h1 {
    font-size: clamp(2rem, 1.5rem + 2.5vw, 3.75rem);
    line-height: 1.15;
  }
  ```
- **Exceptions**: Fixed-size badges, monospace code blocks, and micro-copy.

---

## Rule RESP-05: Container Queries for Component Independence
- **Rule**: Highly reusable card, widget, and feed components SHOULD adapt their layout based on parent container width (`@container`) rather than global viewport width (`@media`).
- **Reason**: A card rendered in a 300px sidebar needs a single-column layout, while the exact same card in an 800px main content area needs a two-column layout—regardless of the screen size.
- **Risk**: Components that break when placed inside split-screen drawers, sidebars, or responsive grid columns.
- **Correct Approach**:
  ```css
  /* ✅ CORRECT */
  .card-container {
    container-type: inline-size;
  }

  .card {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  @container (min-width: 400px) {
    .card {
      flex-direction: row;
      align-items: center;
    }
  }
  ```
- **Exceptions**: High-level page layout grids governed by global viewport boundaries.
