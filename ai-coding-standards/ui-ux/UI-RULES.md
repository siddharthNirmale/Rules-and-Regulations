# User Interface (UI) Design Rules

> **Scope**: Visual hierarchy, spacing systems, typography scales, color harmony, and micro-polish.

---

## Rule UI-01: Strict Adherence to the 4px / 8px Spatial Grid [MUST]
- **Rule**: All margins, paddings, gaps, and component dimensions MUST be multiples of **4px** or **8px** (e.g., `4px`, `8px`, `12px`, `16px`, `24px`, `32px`, `48px`, `64px`). Arbitrary values like `13px`, `27px`, or `37px` are strictly prohibited.
- **Reason**: Mathematical spatial grids create visual rhythm, eliminate subconscious visual clutter, and align elements cleanly across various viewports.
- **Risk**: Chaotic, amateurish layouts that feel disjointed and unpolished.
- **Correct Approach**:
  ```css
  /* ❌ BAD: Arbitrary values */
  margin: 17px; padding: 11px; gap: 7px;

  /* ✅ CORRECT: 8px spatial grid tokens */
  margin: 1rem; /* 16px */
  padding: 0.75rem 1.5rem; /* 12px 24px */
  gap: 1rem; /* 16px */
  ```
- **Exceptions**: Precise 1px borders, subtle box-shadow offsets, or fine icon optical alignment tweaks.

---

## Rule UI-02: Structured Typography Scale & Line Heights [MUST]
- **Rule**: Text sizes and line heights MUST follow a proportional typographic scale. Body text line-height MUST be between **1.4 and 1.6** (relaxed) for readability, while display heading line-height MUST be between **1.1 and 1.25** (tight).
- **Reason**: Default browser line heights cause large display headings to wrap with enormous gaps, while dense body paragraphs become cramped and difficult to read.
- **Risk**: Eye strain, poor scannability, and disproportional visual weights.
- **Correct Approach**:
  ```css
  /* ✅ CORRECT */
  h1 {
    font-size: 2.25rem; /* 36px */
    line-height: 1.15;
    font-weight: 700;
    letter-spacing: -0.025em;
  }

  p {
    font-size: 1rem; /* 16px */
    line-height: 1.5;
    color: #334155;
  }
  ```
- **Exceptions**: Monospace code blocks and compact table data cells.

---

## Rule UI-03: Sophisticated Color Palettes & Contrast Balance [MUST]
- **Rule**: UIs MUST use curated, harmonious color systems with distinct semantic roles (Primary, Neutral, Success, Warning, Error, Surface). Avoid pure saturated RGB primaries (`#ff0000`, `#00ff00`, `#0000ff`) and avoid pure `#000000` text on pure `#ffffff` backgrounds.
- **Reason**: Pure black on pure white causes halation and eye fatigue for astigmatic users. Saturated raw primaries look harsh and amateurish. Deep slates and zincs create premium, calm interfaces.
- **Risk**: Visually aggressive, unpolished interfaces that fatigue users.
- **Correct Approach**:
  ```css
  /* ❌ BAD: Raw harsh primaries */
  background-color: #ff0000; color: #000000;

  /* ✅ CORRECT: Semantic color tokens */
  background-color: #0f172a; /* Slate-900 surface */
  color: #f8fafc; /* Crisp Slate-50 text */
  border-color: #334155; /* Muted Slate-700 border */
  ```
- **Exceptions**: Brand-specific guidelines with pre-established trademark color codes.

---

## Rule UI-04: Multi-Layered, Subtle Elevation (No Harsh Drop Shadows) [SHOULD]
- **Rule**: Box shadows SHOULD be subtle, multi-layered, and natural. Never use single-layer, opaque, harsh black shadows (`box-shadow: 0px 5px 10px #000000`).
- **Reason**: Real-world light produces soft, diffused ambient light combined with subtle directional light. High-opacity shadows look dated and visually overwhelming.
- **Risk**: Heavy, muddy card borders that distract from content.
- **Correct Approach**:
  ```css
  /* ❌ BAD: Harsh, dark shadow */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.5);

  /* ✅ CORRECT: Natural ambient diffusion */
  box-shadow: 
    0 1px 3px 0 rgba(0, 0, 0, 0.05),
    0 4px 6px -1px rgba(0, 0, 0, 0.08),
    0 0 0 1px rgba(0, 0, 0, 0.04);
  ```
- **Exceptions**: Intentional retro/neo-brutalist designs where high-contrast solid borders and sharp drop offsets are the explicit brand aesthetic.
