# User Interface (UI) Design Rules

> **Scope**: Visual hierarchy, spatial grids, typography scales, color harmony, and preventing generic AI UI anti-patterns.

---

## Section 1: Foundational Layout & Visual Systems

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

---

## Section 2: Avoid Common AI UI Mistakes

AI coding agents exhibit well-documented tendencies to generate stereotypical, overly decorated, and generic user interfaces. They frequently reach for the same clichéd aesthetic: dark-mode neon cards, purple-to-cyan gradients, floating dots, pulsing glows, glassmorphism, and excessive nested borders. 

The guidelines in this section establish strict boundaries to ensure interfaces remain **clean, intentional, distinctive, and production-grade**.

### The 6 Core UI Principles

1. **Purpose Over Decoration**: Every visual element, color, border, and animation must serve a functional, informational, or clearly justified aesthetic purpose. Never add styling merely to fill whitespace.
2. **Minimalism Without Blandness**: Create distinctive, polished interfaces through refined typography, intentional spatial rhythm, and strong contrast—not through generic templates or gimmicky visual effects.
3. **Context-Aware Design**: Adapt the visual language to the specific product domain (e.g., enterprise B2B SaaS, developer tooling, financial dashboards, publishing), target audience, and existing design system rather than applying a one-size-fits-all style.
4. **Respect Existing UI**: Never redesign, restyle, or replace an existing interface unless explicitly instructed by the user. Preserve established layouts, color tokens, typography, and interaction patterns when making unrelated changes.
5. **No Arbitrary Additions**: Do not introduce visual elements (icons, badges, particles, dividers) simply because they are ubiquitous in AI-generated prototypes or popular UI component libraries.
6. **Functional Exceptions**: These restrictions are not dogmatic bans. Gradients, borders, animations, and elevation are permitted when they serve an explicit, justified functional requirement (e.g., data visualizations, accessible focus rings, status indicators, or explicit brand design systems).

---

### Rule UI-05: Purpose Over Decoration & Zero Arbitrary Additions [MUST]
- **Rule**: Every visual element (container, divider, icon, badge, background treatment) MUST have an explicit informational or functional justification. Adding visual elements purely as decorative filler is strictly prohibited.
- **Reason**: Decorative noise increases visual cognitive load, distracts users from primary workflows, and dilutes the visual hierarchy.
- **Risk**: Cluttered, amateurish interfaces that feel unfocused and visually overwhelming.
- **Bad vs. Correct Approach**:
  ```tsx
  // ❌ BAD: Arbitrary decorative elements and meaningless badge filler
  <div className="card">
    <div className="flex items-center gap-2">
      <span className="badge-sparkle">✨ AI Powered</span>
      <span className="decorative-dot" />
      <span className="badge-new">NEW</span>
    </div>
    <h2>Dashboard Overview</h2>
    <div className="divider-glow" />
  </div>

  // ✅ CORRECT: Clean, intentional, zero filler
  <div className="card">
    <h2 className="text-xl font-semibold text-slate-900 dark:text-slate-100">
      Dashboard Overview
    </h2>
    <p className="text-sm text-slate-500 mt-1">
      Real-time activity and revenue metrics for your active workspaces.
    </p>
  </div>
  ```
- **Exceptions**: Authentic data-visualization legends and critical status indicators (e.g., green/red operational status dots).

---

### Rule UI-06: Distinctive Layouts (Ban Generic Cookie-Cutter UI) [MUST]
- **Rule**: UIs MUST be designed around the specific information architecture and workflows of the feature. Agents MUST NOT generate generic, predictable templates (e.g., the ubiquitous centered hero section with 3 identical feature cards with circular gradient icons) unless specifically instructed.
- **Reason**: Generic templates make all software look like cookie-cutter AI landing pages, lacking brand identity and failing to optimize for real-world user workflows.
- **Risk**: Poor product differentiation, inefficient use of screen real estate, and mismatched user interactions.
- **Correct Approach**:
  - Analyze the actual data shape: tabular data belongs in tables or data grids, not forced into cards.
  - Prioritize primary tasks: place high-frequency actions directly in the user's primary line of sight.
  - Tailor density: use high density for data-dense dashboards, relaxed density for marketing or reading views.
- **Exceptions**: Standardized scaffolding explicitly dictated by an existing layout framework.

---

### Rule UI-07: Ban Decorative Dots, Particles & Floating Artifacts [MUST NOT]
- **Rule**: AI agents MUST NOT add floating dots, particle canvases, constellation networks, starry mesh backgrounds, glowing grid lines, or random decorative geometric crosshairs to web applications.
- **Reason**: Floating particles and canvas dots consume significant CPU/GPU resources, trigger continuous browser repaint cycles, degrade battery life on mobile devices, and scream "generic AI-generated template".
- **Risk**: Poor Core Web Vitals (INP), high CPU usage, and dated, gimmicky aesthetics.
- **Bad vs. Correct Approach**:
  ```tsx
  // ❌ BAD: Gimmicky floating particles and AI canvas background
  <div className="relative min-h-screen bg-slate-950 overflow-hidden">
    <ParticleBackground particleCount={80} speed={0.5} />
    <div className="absolute top-1/4 left-1/3 w-72 h-72 bg-purple-500/20 rounded-full blur-3xl animate-pulse" />
    <main className="relative z-10">{children}</main>
  </div>

  // ✅ CORRECT: Clean solid surface with semantic elevation
  <div className="min-h-screen bg-slate-50 dark:bg-slate-950 text-slate-900 dark:text-slate-100">
    <main className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">{children}</main>
  </div>
  ```
- **Exceptions**: Interactive particle simulations where physics simulation is the primary functional product feature (e.g., a canvas-based game or physics visualization tool).

---

### Rule UI-08: Purposeful Motion & Ban Pulsing/Continuous Animations [MUST NOT]
- **Rule**: AI agents MUST NOT apply pulsing animations (`animate-pulse`), continuous infinite rotations, glowing breathing effects, or distracting micro-animations to static UI components (cards, buttons, icons, banners). All transitions MUST be snappy (150ms–250ms), purposeful, triggered solely by direct user interaction, and MUST respect `prefers-reduced-motion`.
- **Reason**: Continuous motion draws the user's peripheral vision away from their primary task, creating severe visual fatigue and triggering motion sensitivity issues for neurodivergent and vestibular-disorder users.
- **Risk**: Violations of WCAG 2.2 Success Criterion 2.2.2 (Pause, Stop, Hide), increased rendering overhead, and cognitive fatigue.
- **Bad vs. Correct Approach**:
  ```css
  /* ❌ BAD: Distracting pulsing call-to-action */
  .btn-submit {
    animation: pulse-glow 2s infinite ease-in-out;
  }
  @keyframes pulse-glow {
    0%, 100% { box-shadow: 0 0 5px rgba(99, 102, 241, 0.4); transform: scale(1); }
    50% { box-shadow: 0 0 20px rgba(99, 102, 241, 0.8); transform: scale(1.03); }
  }

  /* ✅ CORRECT: Snappy, user-initiated state transition */
  .btn-submit {
    transition: background-color 150ms ease, transform 100ms ease;
  }
  .btn-submit:hover {
    background-color: var(--color-primary-hover);
  }
  .btn-submit:active {
    transform: scale(0.98);
  }
  @media (prefers-reduced-motion: reduce) {
    .btn-submit {
      transition: none;
    }
  }
  ```
- **Exceptions**: Skeleton loading placeholders during active asynchronous fetching (which may use subtle shimmers), and active progress meters indicating ongoing system operations.

---

### Rule UI-09: Intentional Solid Surfaces (Ban Default Gradients) [MUST NOT]
- **Rule**: AI agents MUST NOT apply gradients as the default styling choice for page backgrounds, cards, buttons, dialogs, or text typography. UIs MUST default to solid, curated color surfaces.
- **Reason**: Multi-stop gradients (especially purple-to-pink or blue-to-violet) reduce text contrast readability, look cliché, and clash with established brand systems.
- **Risk**: Poor color contrast failing WCAG AA (4.5:1), muddy dark-mode rendering, and aesthetic kitsch.
- **Bad vs. Correct Approach**:
  ```css
  /* ❌ BAD: Clichéd multi-stop gradient card and gradient text */
  .card {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  }
  .heading {
    background: linear-gradient(to right, #ec4899, #8b5cf6);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
  }

  /* ✅ CORRECT: High-contrast solid surface with semantic text hierarchy */
  .card {
    background-color: var(--color-surface);
    border: 1px solid var(--color-border);
  }
  .heading {
    color: var(--color-text-primary);
    font-weight: 700;
  }
  ```
- **Exceptions**: Continuous data representations (e.g., temperature/load heatmaps), subtle 2% luminance surface depth in skeuomorphic tokens, or explicit brand logos/illustrations.

---

### Rule UI-10: Spatial Grouping Over Excessive Borders & Nested Boxes [MUST]
- **Rule**: UIs MUST establish visual grouping, separation, and hierarchy through whitespace (`padding`, `margin`, `gap`), background surface shifts, alignment, and typography, rather than wrapping every element, card, column, and input in explicit 1px borders. Avoid nested "box-inside-box-inside-box" containers.
- **Reason**: Outlining every container creates a "cage effect" that segments the screen into rigid visual boxes, making the layout feel cramped, heavy, and fragmented.
- **Risk**: Heavy visual friction, high line density, and degraded scannability.
- **Bad vs. Correct Approach**:
  ```tsx
  // ❌ BAD: Box-inside-box-inside-box with redundant borders
  <div className="border border-slate-300 p-4 rounded-lg">
    <div className="border border-slate-200 p-3 rounded-md mb-3">
      <div className="border border-slate-100 p-2 rounded">
        <span>Nested content</span>
      </div>
    </div>
  </div>

  // ✅ CORRECT: Hierarchy established through spacing and surface contrast
  <div className="bg-white dark:bg-slate-900 rounded-xl p-6 shadow-sm">
    <h3 className="text-lg font-semibold text-slate-900 dark:text-slate-100 mb-4">
      Account Settings
    </h3>
    <div className="space-y-4">
      <div className="bg-slate-50 dark:bg-slate-800/50 rounded-lg p-4">
        <span>Cleanly grouped content</span>
      </div>
    </div>
  </div>
  ```
- **Exceptions**: Form input boundaries, high-contrast accessibility modes, and standalone data tables where cell borders are necessary to distinguish dense numeric rows.

---

### Rule UI-11: Restrict Decorative Effects (Glows, Blurs, Glassmorphism, Capsules) [MUST NOT]
- **Rule**: AI agents MUST NOT apply heavy drop shadows, neon box-glows, glassmorphism (`backdrop-filter: blur`), frosted glass overlays, or wrap regular text in capsule pills (`rounded-full` border badges) unless explicitly requested.
- **Reason**: Glassmorphism and backdrop blurs cause severe GPU compositing overhead on low-end devices, degrade text contrast depending on underlying scrolled content, and look dated.
- **Risk**: Severe frame drops (jank) during scrolling, contrast failure on dynamic backgrounds, and visual gimmicks.
- **Bad vs. Correct Approach**:
  ```css
  /* ❌ BAD: Heavy glassmorphism, blur, and neon glow */
  .card-modal {
    background: rgba(255, 255, 255, 0.05);
    backdrop-filter: blur(20px);
    border: 1px solid rgba(255, 255, 255, 0.1);
    box-shadow: 0 0 30px rgba(139, 92, 246, 0.3);
  }

  /* ✅ CORRECT: Clean solid surface with subtle, performant elevation */
  .card-modal {
    background-color: var(--color-surface-elevated);
    border: 1px solid var(--color-border-subtle);
    box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -4px rgba(0, 0, 0, 0.1);
  }
  ```
- **Exceptions**: Fixed floating navigation headers that require slight backdrop blurring to prevent underlying scrolled text collisions, or native OS-level translucent sidebars (e.g., macOS/iOS native shells).

---

### Rule UI-12: Eliminate Visual Clutter (No Filler Icons, Badges, or Dividers) [MUST NOT]
- **Rule**: AI agents MUST NOT add icons to every button, heading, or list item simply to fill empty space. Avoid inserting horizontal divider rules (`<hr>`) between every paragraph or section when whitespace suffices.
- **Reason**: When every element has an icon, no element stands out. Icons lose their utility as visual wayfinders and become cognitive visual noise.
- **Risk**: Cognitive overload, slower scanning speeds, and messy visual hierarchy.
- **Correct Approach**:
  - Use icons only when they provide instant recognition (e.g., search magnifying glass, close 'X', external link indicator, chevron for dropdowns).
  - Use whitespace margins (`gap`, `margin-bottom`) to separate sections; reserve `<hr>` dividers strictly for major thematic breaks.
- **Exceptions**: Icon-only utility buttons with mandatory `aria-label` tooltips.

---

### Rule UI-13: Ban Stereotypical AI Design Clichés [MUST NOT]
- **Rule**: AI agents MUST NOT inject "AI-themed" visual clichés into standard software applications:
  - Sparkle icons (`✨`, `SparklesIcon`) on features that do not involve machine learning.
  - Dark-mode-only futuristic cyberpunk motifs (electric purple, neon cyan, dark slate-950) for normal business software.
  - "Robot", "brain", or "magic wand" iconography for standard automation rules or search inputs.
  - Simulated typing animations or fake delays on standard deterministic computations.
- **Reason**: Treating every feature as an ethereal "magical AI" experience degrades professional software into gimmickry and causes user skepticism.
- **Risk**: Loss of professional product credibility and user annoyance with sluggish simulated experiences.
- **Exceptions**: Products whose core domain is explicitly generative AI tooling and where sparkle branding is the established system standard.

---

### Rule UI-14: Respect Existing Interfaces & Avoid Unsolicited Redesigns [MUST]
- **Rule**: When tasked with modifying, fixing, or extending an existing UI, the AI agent MUST strictly preserve the existing design language, layout structure, color tokens, typography, and component styling. The agent MUST NOT opportunistically restyle or "modernize" adjacent components unless explicitly instructed.
- **Reason**: Unsolicited UI changes clash with established product conventions, break user muscle memory, and create massive review diffs.
- **Risk**: High PR rejection rates, brand inconsistency, and customer frustration.
- **Exceptions**: Fixing an objective accessibility bug (e.g., correcting an unreadable 2:1 contrast ratio to meet WCAG AA 4.5:1).

---

### Rule UI-15: Radical Simplicity Over Unnecessary UI Complexity [MUST]
- **Rule**: AI agents MUST prefer the simplest, most lightweight CSS and component implementation that achieves the design goal.
  - Do not install heavy external UI or animation libraries (Framer Motion, GSAP, Lottie) for interactions achievable with standard CSS `transition` or `animation`.
  - Do not wrap simple text elements in 4 layers of nested `div` containers.
  - Do not introduce complex stateful drawer/accordion/modal widgets when a clean, visible, single-page layout is more usable.
- **Reason**: Unnecessary UI complexity bloats bundle size, degrades frame rates, increases the maintenance burden, and introduces fragile responsive edge cases.
- **Risk**: Slow First Input Delay / INP, fragile responsive layouts, and hard-to-maintain codebases.
- **Exceptions**: Complex data visualization workflows (e.g., canvas chart rendering) where standard HTML/CSS cannot meet functional requirements.

