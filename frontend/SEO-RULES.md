# Search Engine Optimization (SEO) Rules

> **Scope**: Search engine indexing, social graph discovery, semantic markup, and metadata hygiene.

---

## Rule SEO-01: Unique Title Tags and Meta Descriptions per Route [MUST]
- **Rule**: Every publicly accessible route MUST render a unique, descriptive `<title>` tag and `<meta name="description">` tag. The `<title>` MUST follow the format `"Page Title | Brand Name"` (under 60 characters), and the meta description MUST accurately summarize the page content (between 120 and 160 characters).
- **Reason**: Search engine result pages (SERPs) use title and description tags to generate search snippets. Missing or duplicated titles degrade click-through rates and search ranking.
- **Risk**: Search engines ignoring duplicate titles or automatically generating awkward truncated snippets.
- **Correct Approach**:
  ```tsx
  // Next.js Metadata API example
  export const metadata: Metadata = {
    title: 'Automated Cloud Telemetry & Observability | CloudPulse',
    description: 'Monitor microservice latency, trace distributed HTTP requests, and prevent cloud infrastructure downtime with CloudPulse analytics platform.',
    alternates: {
      canonical: 'https://cloudpulse.io/platform/observability'
    }
  };
  ```
- **Exceptions**: Authenticated internal dashboards behind login screens (`robots: { index: false }`).

---

## Rule SEO-02: Strictly Single `<h1>` and Sequential Heading Hierarchy [MUST]
- **Rule**: Every web page MUST render exactly one `<h1>` element representing the primary topic of the page. Subsections MUST follow sequential heading levels (`<h2>` under `<h1>`, `<h3>` under `<h2>`). Skipping levels (e.g. going from `<h1>` directly to `<h4>`) is strictly prohibited.
- **Reason**: Search engine spiders (Googlebot, Bingbot) and screen readers use heading trees to build an outline of the document's topical hierarchy.
- **Risk**: Search indexing penalties and broken screen reader outline navigation.
- **Correct Approach**:
  ```html
  <!-- ✅ CORRECT: Strict sequential hierarchy -->
  <main>
    <h1>Infrastructure Monitoring</h1>
    <section>
      <h2>Real-Time Metrics</h2>
      <section>
        <h3>CPU & Memory Utilization</h3>
      </section>
      <section>
        <h3>Disk I/O IOPS</h3>
      </section>
    </section>
  </main>
  ```
- **Exceptions**: None on public marketing and content pages.

---

## Rule SEO-03: Mandatory OpenGraph & Social Card Tags [SHOULD]
- **Rule**: Public pages SHOULD include OpenGraph (`og:title`, `og:description`, `og:image`, `og:url`, `og:type`) and Twitter Card (`twitter:card: summary_large_image`) metadata tags.
- **Reason**: When users share links on LinkedIn, Slack, Twitter/X, or Discord, platforms crawl these tags to generate rich social media preview cards.
- **Risk**: Broken, missing, or distorted preview cards that sharply decrease click-through traffic.
- **Correct Approach**:
  ```html
  <meta property="og:title" content="Real-Time Cloud Telemetry | CloudPulse">
  <meta property="og:description" content="Monitor microservices and prevent outages with automated tracing.">
  <meta property="og:image" content="https://cloudpulse.io/images/og/telemetry.jpg">
  <meta property="og:url" content="https://cloudpulse.io/platform/telemetry">
  <meta property="og:type" content="website">
  <meta name="twitter:card" content="summary_large_image">
  ```
- **Exceptions**: Private user admin panels or ephemeral checkout pages.

---

## Rule SEO-04: Server-Side Rendering (SSR) or Static Site Generation (SSG) for Public Content [MUST]
- **Rule**: Content intended for public search engine discovery MUST NOT be rendered exclusively on the client behind client-side-only JavaScript fetches (Pure Single Page Apps). Public content MUST use Server-Side Rendering (SSR), Static Site Generation (SSG), or Incremental Static Regeneration (ISR).
- **Reason**: While modern search crawlers can execute JavaScript, crawler execution budgets are limited and delayed, often leading to stale indexes or failure to index dynamically rendered content.
- **Risk**: Incomplete search indexing and massive loss of organic search traffic.
- **Correct Approach**: Use frameworks supporting pre-rendering (Next.js, Nuxt, Astro, SvelteKit) for public-facing blogs, landing pages, and product catalogs.
- **Exceptions**: Gated SaaS applications behind mandatory user login screens.

---

## Rule SEO-05: Valid Canonical URLs & Crawl Directives [MUST]
- **Rule**: Every public page MUST include a `<link rel="canonical" href="...">` tag referencing its authoritative URL, and the root domain MUST serve a valid `robots.txt` and `sitemap.xml`.
- **Reason**: E-commerce and filtering queries (`?category=shoes&sort=price` vs `?sort=price&category=shoes`) generate multiple URLs with duplicate content. Canonical tags instruct search engines which URL to rank, preventing duplicate content penalties.
- **Risk**: Diluted search authority and lower domain ranking.
- **Correct Approach**:
  ```html
  <!-- In <head> -->
  <link rel="canonical" href="https://example.com/products/running-shoes" />
  ```
  ```text
  # robots.txt
  User-agent: *
  Allow: /
  Disallow: /api/
  Disallow: /admin/
  Sitemap: https://example.com/sitemap.xml
  ```
- **Exceptions**: Non-production staging environments (which MUST return `X-Robots-Tag: noindex, nofollow` to prevent indexing of test domains).
