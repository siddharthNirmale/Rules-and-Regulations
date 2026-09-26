# Frontend Performance & Core Web Vitals Rules

> **Objective**: Guarantee production web applications achieve fast loading, low latency, and zero visual jank across desktop and mobile devices.

---

## Performance Thresholds (Core Web Vitals)
All frontend production deliverables MUST target and maintain:
- **Largest Contentful Paint (LCP)**: <= 2.5 seconds (Good)
- **Interaction to Next Paint (INP)**: <= 200 milliseconds (Good)
- **Cumulative Layout Shift (CLS)**: <= 0.1 (Good)

---

## Rule PERF-01: Prevent Cumulative Layout Shift with Explicit Dimensions
- **Rule**: All `<img>`, `<video>`, `<iframe>`, and ad containers MUST declare explicit `width` and `height` attributes or CSS `aspect-ratio`.
- **Reason**: The browser uses the aspect ratio to allocate layout space before the media asset finishes downloading. Without dimensions, the layout jumps when the asset arrives.
- **Risk**: High CLS score, causing user misclicks on buttons while content shifts unpredictably.
- **Correct Approach**:
  ```html
  <!-- ❌ BAD: No dimensions, causes layout shift on load -->
  <img src="/hero.jpg" alt="Platform dashboard" />

  <!-- ✅ CORRECT: Aspect ratio reserved prior to load -->
  <img
    src="/hero.webp"
    alt="Platform dashboard"
    width="1200"
    height="630"
    loading="lazy"
    decoding="async"
    class="w-full h-auto aspect-[1200/630]"
  />
  ```
- **Exceptions**: Hero images above the fold MUST use `loading="eager"` or `fetchpriority="high"`, never `loading="lazy"`.

---

## Rule PERF-02: Route-Level Code Splitting & Dynamic Imports
- **Rule**: Non-critical routes, large modals, rich-text editors, and charting libraries MUST be loaded asynchronously via dynamic `import()` / lazy loading.
- **Reason**: Bundling heavy third-party libraries (e.g. Monaco Editor, Chart.js, PDF renderers) into the main bundle inflates the initial payload by hundreds of kilobytes.
- **Risk**: Sluggish initial page load (high LCP) and long JavaScript parse/compile times on low-end mobile devices.
- **Correct Approach**:
  ```tsx
  // ❌ BAD: Direct static import bloats the critical bundle
  import HeavyChart from './HeavyChart';

  // ✅ CORRECT: Asynchronous lazy import loaded on demand
  import { lazy, Suspense } from 'react';
  const HeavyChart = lazy(() => import('./HeavyChart'));

  function AnalyticsView() {
    return (
      <Suspense fallback={<ChartSkeleton />}>
        <HeavyChart />
      </Suspense>
    );
  }
  ```
- **Exceptions**: Primary layout shells and components strictly required for initial render of the current view.

---

## Rule PERF-03: Virtualize Collections Exceeding 100 Elements
- **Rule**: Any dataset rendered as a list or table exceeding 100 rows MUST utilize windowing / virtualization (e.g., `@tanstack/react-virtual`) rather than mounting hundreds of DOM nodes.
- **Reason**: Creating thousands of DOM nodes causes heavy memory allocation, degrades browser scroll performance to sub-30 FPS, and crashes mobile browsers.
- **Risk**: Stuttering UI, slow interactions, and memory crashes.
- **Correct Approach**:
  ```tsx
  // ✅ CORRECT
  import { useVirtualizer } from '@tanstack/react-virtual';

  function VirtualUserList({ users }) {
    const parentRef = useRef<HTMLDivElement>(null);
    const rowVirtualizer = useVirtualizer({
      count: users.length,
      getScrollElement: () => parentRef.current,
      estimateSize: () => 48,
      overscan: 5
    });

    return (
      <div ref={parentRef} className="h-96 overflow-auto">
        <div style={{ height: `${rowVirtualizer.getTotalSize()}px`, position: 'relative' }}>
          {rowVirtualizer.getVirtualItems().map(virtualRow => (
            <div
              key={virtualRow.key}
              style={{
                position: 'absolute',
                top: 0,
                transform: `translateY(${virtualRow.start}px)`,
                height: `${virtualRow.size}px`
              }}
            >
              {users[virtualRow.index].name}
            </div>
          ))}
        </div>
      </div>
    );
  }
  ```
- **Exceptions**: Short collections guaranteed to remain under 100 elements.

---

## Rule PERF-04: Debounce or Throttle High-Frequency Events
- **Rule**: Handlers attached to `scroll`, `resize`, `mousemove`, or search inputs triggering asynchronous queries MUST be throttled or debounced.
- **Reason**: Firing heavy handlers or network requests on every keystroke or pixel scroll fires dozens of events per second, exhausting CPU time and network connections.
- **Risk**: High Interaction to Next Paint (INP) latency and rate-limiting from backend API servers.
- **Correct Approach**:
  ```typescript
  // ✅ CORRECT
  import { debounce } from './utils/debounce';

  const handleSearch = useMemo(
    () => debounce((query: string) => {
      fetchSearchResults(query);
    }, 300),
    []
  );
  ```
- **Exceptions**: Direct CSS manipulations driven by modern CSS properties like `scroll-driven animations` or `ResizeObserver` with lightweight logic.

---

## Rule PERF-05: Web Font Optimization & Layout Shift Prevention
- **Rule**: Custom web fonts MUST specify `font-display: swap` or `font-display: optional`, self-host font files in WOFF2 format, and preload primary body font subsets.
- **Reason**: Relying on external third-party font CDNs adds extra DNS, TLS, and network hops. Without `font-display: swap`, text remains invisible (FOIT - Flash of Invisible Text) while fonts download.
- **Risk**: Prolonged invisible text delays First Contentful Paint and frustrates readers.
- **Correct Approach**:
  ```html
  <!-- ✅ CORRECT: Preload primary font subset in woff2 format -->
  <link
    rel="preload"
    href="/fonts/inter-latin.woff2"
    as="font"
    type="font/woff2"
    crossorigin
  />
  ```
  ```css
  @font-face {
    font-family: 'Inter';
    src: url('/fonts/inter-latin.woff2') format('woff2');
    font-display: swap;
    font-weight: 100 900;
  }
  ```
- **Exceptions**: System font stacks (`system-ui`, `-apple-system`, `sans-serif`) which require zero network downloads.
