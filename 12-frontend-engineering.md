# 🎨 Frontend Engineering — Production Engineering Guide

> How to build fast, accessible, secure, and maintainable web frontends: platform fundamentals, choosing frameworks and rendering strategies (CSR, SSR, SSG, ISR, streaming, islands, React Server Components), project structure, TypeScript, state and data fetching, forms, styling and design systems, accessibility (WCAG 2.2), performance and Core Web Vitals, images and fonts, internationalisation, browser security (CSP, XSS, auth tokens), testing, build tooling, observability, SEO, PWAs/offline, micro-frontends, deployment, and dependency/upgrade management.
>
> Related: [05 UI/UX & accessibility design §21](./05-software-design.md#21-ui-ux-and-accessibility-design) · [06 JS/TS standards §8](./06-coding-standards.md#8-javascript--typescript) · [08 E2E, a11y, visual testing](./08-testing-and-quality.md) · [09 Browser security §11–12](./09-security-engineering.md) · [11 API consumption](./11-api-and-integration.md) · [19 Performance](./19-performance-and-scalability.md) · IDE: [32 VS Code extensions](./32-vscode-idea-extensions.md)

---

## 📚 Table of Contents

- [🎨 Frontend Engineering — Production Engineering Guide](#-frontend-engineering--production-engineering-guide)
  - [📚 Table of Contents](#-table-of-contents)
  - [1. Principles](#1-principles)
  - [2. Web Platform Fundamentals](#2-web-platform-fundamentals)
    - [2.1 How a page loads (critical rendering path)](#21-how-a-page-loads-critical-rendering-path)
    - [2.2 Semantic HTML first](#22-semantic-html-first)
    - [2.3 Browser support policy](#23-browser-support-policy)
  - [3. Choosing a Framework](#3-choosing-a-framework)
    - [3.1 Landscape (as of late 2026 — verify current major versions before starting)](#31-landscape-as-of-late-2026--verify-current-major-versions-before-starting)
    - [3.2 Decision guide](#32-decision-guide)
  - [4. Rendering Strategies](#4-rendering-strategies)
  - [5. React Server Components and Modern React](#5-react-server-components-and-modern-react)
    - [5.1 Server vs Client Components (React 19 / Next.js App Router)](#51-server-vs-client-components-react-19--nextjs-app-router)
    - [5.2 Server code rules](#52-server-code-rules)
    - [5.3 React 19.x features worth knowing](#53-react-19x-features-worth-knowing)
  - [6. Project Structure](#6-project-structure)
    - [6.1 Feature-based structure (example, Next.js App Router)](#61-feature-based-structure-example-nextjs-app-router)
    - [6.2 Rules](#62-rules)
  - [7. TypeScript in the Frontend](#7-typescript-in-the-frontend)
  - [8. State Management](#8-state-management)
    - [8.1 Classify state first](#81-classify-state-first)
    - [8.2 Rules](#82-rules)
  - [9. Data Fetching and API Integration](#9-data-fetching-and-api-integration)
    - [9.1 Patterns](#91-patterns)
    - [9.2 TanStack Query example](#92-tanstack-query-example)
    - [9.3 Rules](#93-rules)
  - [10. Forms and Validation](#10-forms-and-validation)
  - [11. Styling and Design Systems](#11-styling-and-design-systems)
    - [11.1 Styling approaches](#111-styling-approaches)
    - [11.2 Design system architecture](#112-design-system-architecture)
    - [11.3 Responsive design](#113-responsive-design)
  - [12. Accessibility (WCAG 2.2)](#12-accessibility-wcag-22)
    - [12.1 Engineering checklist](#121-engineering-checklist)
    - [12.2 Testing](#122-testing)
  - [13. Performance and Core Web Vitals](#13-performance-and-core-web-vitals)
    - [13.1 Core Web Vitals](#131-core-web-vitals)
    - [13.2 Field vs lab data](#132-field-vs-lab-data)
    - [13.3 Fixes by metric](#133-fixes-by-metric)
    - [13.4 Performance budgets (enforce in CI)](#134-performance-budgets-enforce-in-ci)
    - [13.5 Bundle discipline](#135-bundle-discipline)
  - [14. Images, Fonts, and Media](#14-images-fonts-and-media)
    - [14.1 Images](#141-images)
    - [14.2 Fonts](#142-fonts)
    - [14.3 Video](#143-video)
  - [15. Internationalisation and Localisation](#15-internationalisation-and-localisation)
  - [16. Frontend Security](#16-frontend-security)
  - [17. Authentication in the Browser](#17-authentication-in-the-browser)
  - [18. Testing the Frontend](#18-testing-the-frontend)
  - [19. Build Tooling](#19-build-tooling)
  - [20. Observability and Error Tracking](#20-observability-and-error-tracking)
  - [21. SEO and Social Sharing](#21-seo-and-social-sharing)
  - [22. PWAs, Offline, and Service Workers](#22-pwas-offline-and-service-workers)
  - [23. Micro-Frontends](#23-micro-frontends)
  - [24. Deployment and Delivery](#24-deployment-and-delivery)
  - [25. Dependency and Upgrade Management](#25-dependency-and-upgrade-management)
  - [26. Checklists](#26-checklists)
    - [New frontend project](#new-frontend-project)
    - [Every UI PR](#every-ui-pr)
    - [Release](#release)
  - [27. References](#27-references)
    - [Platform and standards](#platform-and-standards)
    - [Frameworks and tools](#frameworks-and-tools)
    - [Security](#security)

---

## 1. Principles

| # | Principle |
|---:|---|
| 1 | **Users first, on real devices and networks** — test on mid-range Android phones and slow 4G, not only on a developer MacBook on fibre. |
| 2 | **Use the platform** — semantic HTML, native form controls, CSS, and browser APIs before libraries. |
| 3 | **Accessibility is not optional** — WCAG 2.2 AA is the baseline (and often a legal requirement). |
| 4 | **Ship less JavaScript** — every kilobyte costs download, parse, and execution time on low-end devices. |
| 5 | **Server is the source of truth** — client state caches server state; don't duplicate it carelessly. |
| 6 | **Never trust the browser** — validation and authorisation happen on the server; the frontend is untrusted code. |
| 7 | **Measure with field data** — Core Web Vitals from real users decide performance work. |
| 8 | **Progressive enhancement** — core tasks should degrade gracefully when JS fails or loads slowly. |

---

## 2. Web Platform Fundamentals

### 2.1 How a page loads (critical rendering path)

```text
DNS → TCP/QUIC → TLS → HTTP request → HTML streamed
  → parse HTML → discover CSS/JS/fonts/images (preload scanner)
  → CSSOM + DOM → render tree → layout → paint → composite
  → JS downloads, parses, executes (can block parsing if not async/defer/module)
  → hydration (frameworks attach event handlers) → interactive
```

| Optimisation lever | How |
|---|---|
| Reduce round trips | HTTP/2 or HTTP/3, CDN, `preconnect` to critical origins |
| Unblock rendering | Inline critical CSS (when worthwhile), `defer`/`type="module"` scripts |
| Prioritise | `fetchpriority="high"` on LCP image, `preload` key resources |
| Reduce bytes | Compression (Brotli), code splitting, tree shaking, modern image formats |
| Reduce main-thread work | Less JS, break up long tasks, web workers |

### 2.2 Semantic HTML first

```html
<!-- ❌ div soup -->
<div class="btn" onclick="submit()">Pay</div>

<!-- ✅ native semantics: keyboard, focus, screen reader, form submission for free -->
<button type="submit">Pay ₹2,499</button>

<!-- Landmarks -->
<header>…</header>
<nav aria-label="Main">…</nav>
<main id="main">…</main>
<footer>…</footer>
```

### 2.3 Browser support policy

Define it explicitly (e.g. **Baseline "widely available"** features plus your analytics-driven browser list) and encode it in `browserslist` so tooling transpiles/polyfills consistently.

```text
# .browserslistrc (example — derive from your analytics)
> 0.5% in IN
last 2 Chrome versions
last 2 Safari versions
last 2 Firefox versions
last 2 Edge versions
iOS >= 16
not dead
```

Web Platform **Baseline** (from the WebDX Community Group, shown on MDN and web.dev) tells you whether a feature is "newly available" or "widely available" across major browsers.

---

## 3. Choosing a Framework

### 3.1 Landscape (as of late 2026 — verify current major versions before starting)

| Framework / meta-framework | Model | Strengths | Consider when |
|---|---|---|---|
| **React 19.x** + **Next.js** (16+) | Components, Server Components, server actions; Next.js adds routing, SSR/SSG, caching | Largest ecosystem and hiring pool; RSC; React Compiler (stable since Oct 2025) for automatic memoisation | Product web apps, SaaS, content + app hybrids |
| React + **React Router** (framework mode) / **TanStack Start** | React with routing-centric full-stack framework | Web-standards-oriented data loading | Teams wanting less framework magic than Next.js |
| **Vue 3** + **Nuxt** | Reactive templates, Composition API | Gentle learning curve, cohesive ecosystem | Teams preferring templates; progressive adoption |
| **Angular** (signals, standalone components) | Opinionated full framework, DI, TypeScript-first | Enterprise consistency, batteries included | Large enterprise teams, long-lived apps |
| **Svelte 5** + **SvelteKit** | Compiler, runes-based reactivity | Small bundles, simple mental model | Performance-sensitive apps, smaller teams |
| **Astro** | Content-first, islands architecture | Ships zero JS by default; mix frameworks | Marketing sites, docs, blogs, content-heavy sites |
| **SolidJS / Qwik** | Fine-grained reactivity / resumability | Performance | Niche/performance-critical cases |
| HTMX / Hotwire / Livewire | Server-rendered HTML with partial updates | Minimal JS, backend-centric teams | CRUD/admin apps with server-side teams |

### 3.2 Decision guide

```text
Content site (marketing, docs, blog)?            → Astro (or SSG with your framework)
Product app, SEO matters, React team?            → Next.js (App Router) or React Router framework mode
Internal admin/back-office, backend-heavy team?  → HTMX/Livewire/Hotwire or a SPA with a component library
Large enterprise, many teams, strict conventions? → Angular (or React with strong internal platform)
Embedded widget / very small bundle?             → Svelte, Preact, or vanilla Web Components
```

> Choose the framework your team can operate and hire for. Record the decision in an ADR (chapter 04 §9).

---

## 4. Rendering Strategies

| Strategy | HTML produced | Pros | Cons | Use for |
|---|---|---|---|---|
| **CSR** (client-side rendering, SPA) | In the browser | Simple hosting; rich interactivity | Slow first load, SEO depends on crawler JS | Logged-in dashboards, internal tools |
| **SSR** (server-side rendering) | Per request on server | Fast first paint, SEO, fresh data | Server cost; hydration cost | Personalised pages needing SEO |
| **SSG** (static site generation) | At build time | Fastest, cheapest (CDN) | Rebuild for changes | Docs, marketing, blogs |
| **ISR / revalidation** | Static, regenerated on interval/on-demand | Static speed + freshness | Cache invalidation complexity | Catalogues, content with periodic updates |
| **Streaming SSR** | Server streams HTML chunks with Suspense boundaries | Early bytes, progressive display | Framework support required | Data-heavy pages |
| **Partial prerendering / cache components** | Static shell + dynamic holes | Static speed with dynamic parts | Newer patterns, framework-specific | Product pages with personalised bits |
| **Islands** (Astro) | Static HTML, hydrate only interactive islands | Minimal JS | Less suited to app-like UIs | Content sites with some interactivity |
| **Edge rendering** | At CDN edge | Low latency globally | Runtime limits, data locality | Personalisation, geo-specific content |

```text
Rule of thumb: render as much as possible on the server/at build time, hydrate as little as possible,
and stream when data is slow.
```

---

## 5. React Server Components and Modern React

### 5.1 Server vs Client Components (React 19 / Next.js App Router)

| | Server Components (default in App Router) | Client Components (`'use client'`) |
|---|---|---|
| Runs | On the server (build or request) | On the server for SSR **and** in the browser |
| Can | Fetch data directly, read secrets, use server-only libraries | Use state, effects, event handlers, browser APIs |
| Ships JS to client | No (only the rendered result) | Yes |
| Use for | Data fetching, layouts, static content | Interactivity |

```tsx
// app/orders/[id]/page.tsx — Server Component
import { getOrder } from '@/server/orders';          // server-only module
import { RetryPaymentButton } from './retry-payment-button';

export default async function OrderPage({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const order = await getOrder(id);                    // authorise inside getOrder!
  return (
    <main>
      <h1>Order {order.id}</h1>
      <p>Status: {order.status}</p>
      {order.status === 'payment_failed' && <RetryPaymentButton orderId={order.id} />}
    </main>
  );
}
```

```tsx
// retry-payment-button.tsx — Client Component
'use client';
import { useTransition } from 'react';
import { retryPayment } from './actions';

export function RetryPaymentButton({ orderId }: { orderId: string }) {
  const [pending, startTransition] = useTransition();
  return (
    <button disabled={pending} onClick={() => startTransition(() => retryPayment(orderId))}>
      {pending ? 'Retrying…' : 'Retry payment'}
    </button>
  );
}
```

```ts
// actions.ts — Server Action
'use server';
import { requireSession } from '@/server/auth';
import { z } from 'zod';

export async function retryPayment(rawOrderId: string) {
  const session = await requireSession();                         // authenticate
  const orderId = z.string().regex(/^ord_[a-z0-9]{10}$/).parse(rawOrderId);  // validate
  await paymentsService.retry({ orderId, customerId: session.userId });     // authorise inside service
}
```

### 5.2 Server code rules

```text
🔴 Server Actions and route handlers are PUBLIC HTTP endpoints — authenticate, validate, and authorise in every one
🔴 Mark server-only modules (e.g. import 'server-only') so secrets never end up in client bundles
🔴 Never pass secrets or full DB entities as props to Client Components (they are serialised to the browser)
🔴 Keep frameworks patched — critical vulnerabilities have been disclosed in server-component/action
    implementations; subscribe to framework security advisories and upgrade promptly
🟠 Use the framework's caching APIs deliberately; document what is cached, for how long, and per-user vs shared
```

### 5.3 React 19.x features worth knowing

| Feature | Use |
|---|---|
| **Actions**, `useActionState`, `useFormStatus`, `useOptimistic` | Forms and mutations with pending/optimistic states |
| `use()` | Read promises/context in render (with Suspense) |
| **React Compiler** (stable v1, Oct 2025) | Automatic memoisation — reduces manual `useMemo`/`useCallback` |
| `<Activity>` (19.2) | Keep hidden UI mounted with preserved state, deprioritised |
| `useEffectEvent` (19.2) | Stable event callbacks inside effects without re-running them |
| Document metadata in components | `<title>`, `<meta>` rendered anywhere and hoisted |
| `ref` as a prop | `forwardRef` no longer needed for function components |

---

## 6. Project Structure

### 6.1 Feature-based structure (example, Next.js App Router)

```text
src/
├── app/                         # routes (thin: compose features)
│   ├── (marketing)/
│   ├── (shop)/checkout/page.tsx
│   ├── orders/[id]/page.tsx
│   ├── api/                     # route handlers (webhooks, BFF endpoints)
│   └── layout.tsx
├── features/                    # vertical slices
│   ├── checkout/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── api.ts               # typed API client calls for this feature
│   │   ├── schema.ts            # zod schemas
│   │   └── checkout.test.tsx
│   └── orders/
├── components/ui/               # design-system primitives (Button, Dialog, Input…)
├── lib/                         # framework-agnostic utilities (formatting, money, dates)
├── server/                      # server-only code (auth, data access) — import 'server-only'
├── styles/
└── test/                        # test utilities, MSW handlers, fixtures
```

### 6.2 Rules

```text
🔴 Features don't import from other features' internals — share via components/ui or lib, or a public index
🟠 Co-locate tests, stories, and styles with components
🟠 Enforce boundaries with ESLint import rules / dependency-cruiser / Nx module boundaries
🟠 Keep route files thin; logic lives in features
```

---

## 7. TypeScript in the Frontend

```text
🔴 strict mode (chapter 06 §8.1); no `any` in application code
🔴 Generate API types from contracts (openapi-typescript, Orval, GraphQL Codegen) — never hand-copy DTOs
🔴 Validate untrusted data at runtime (API responses from third parties, URL params, localStorage, postMessage)
🟠 Discriminated unions for UI states: { status: 'loading' } | { status: 'error', error } | { status: 'success', data }
🟠 Branded types for IDs and money to prevent mix-ups
```

```ts
type RemoteData<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'error'; error: Error }
  | { status: 'success'; data: T };

type OrderId = string & { readonly __brand: 'OrderId' };
```

---

## 8. State Management

### 8.1 Classify state first

| State type | Example | Best home |
|---|---|---|
| **Server/remote state** | Orders, products, user profile | Data-fetching cache (TanStack Query, SWR, RTK Query, Apollo/urql) or server components |
| **URL state** | Filters, sort, pagination, selected tab | Router search params — shareable, bookmarkable |
| **Form state** | Inputs, validation errors | Form library or native forms + actions |
| **Local UI state** | Open/closed, hover, input focus | Component state |
| **Global client state** | Theme, feature flags, cart (pre-checkout), auth session info | Small store (Zustand, Jotai, Redux Toolkit, Pinia, Angular signals/NgRx SignalStore) or context |
| **Persistent client state** | Draft content, preferences | `localStorage`/IndexedDB with schema version + validation |

> Most "state management problems" are server-state caching problems. Use a data-fetching library before reaching for a global store.

### 8.2 Rules

```text
🔴 Never store secrets or access tokens in localStorage/sessionStorage (XSS-readable) — §17
🟠 Derive, don't duplicate: compute values from source state instead of syncing copies
🟠 Keep global stores small and typed; avoid one giant store
🟠 Put shareable view state in the URL
```

---

## 9. Data Fetching and API Integration

### 9.1 Patterns

| Pattern | Use |
|---|---|
| Server Components / loaders fetch on the server | Initial page data, SEO |
| Query library (TanStack Query, SWR) | Client-side caching, revalidation, retries, pagination, optimistic updates |
| BFF (Backend for Frontend) | Aggregate services, hold tokens server-side, tailor payloads (chapter 11 §18.3) |
| Typed clients from OpenAPI/GraphQL | Compile-time safety |

### 9.2 TanStack Query example

```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

export function useOrder(orderId: string) {
  return useQuery({
    queryKey: ['order', orderId],
    queryFn: ({ signal }) => api.getOrder(orderId, { signal }),   // abortable
    staleTime: 30_000,
  });
}

export function useRetryPayment(orderId: string) {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: (method: 'upi' | 'netbanking') =>
      api.retryPayment(orderId, method, { idempotencyKey: crypto.randomUUID() }),
    onSuccess: () => qc.invalidateQueries({ queryKey: ['order', orderId] }),
  });
}
```

### 9.3 Rules

```text
🔴 Handle loading, empty, error, and partial states for every data view
🔴 Abort in-flight requests on unmount/navigation (AbortController)
🔴 Show user-friendly errors; log details with correlation IDs (chapter 11 §5)
🟠 Avoid request waterfalls: fetch in parallel; move fetching up (loaders/server components)
🟠 Idempotency keys for payments and other retried mutations
🟠 Mock APIs with MSW in development and tests
```

---

## 10. Forms and Validation

```text
🔴 Use <form>, <label for>, native input types (email, tel, number with care, date), autocomplete attributes
🔴 Client-side validation is UX only — always validate on the server
🔴 Share schemas between client and server where possible (zod/valibot/yup)
🔴 Errors: announced to screen readers (aria-live / aria-describedby), shown next to fields, plus a summary for long forms
🟠 Don't disable the submit button as the only feedback; show why
🟠 Preserve user input on errors; never clear the form
🟠 Correct `inputmode` and `autocomplete` (e.g. autocomplete="one-time-code" for OTP, "postal-code", "cc-number")
```

```tsx
<form action={submitAddress} noValidate>
  <label htmlFor="pin">PIN code</label>
  <input id="pin" name="pin" inputMode="numeric" autoComplete="postal-code"
         aria-invalid={!!errors.pin} aria-describedby={errors.pin ? 'pin-error' : undefined}
         required pattern="[1-9][0-9]{5}" />
  {errors.pin && <p id="pin-error" role="alert">Enter a valid 6-digit PIN code.</p>}
  <button type="submit">Save address</button>
</form>
```

Form libraries: React Hook Form, TanStack Form, Conform (progressive enhancement), Angular Reactive Forms, VeeValidate (Vue), Superforms (SvelteKit).

---

## 11. Styling and Design Systems

### 11.1 Styling approaches

| Approach | Pros | Cons |
|---|---|---|
| **Utility-first CSS** (Tailwind CSS) | Fast, consistent tokens, small CSS output | Verbose markup; needs component abstractions |
| **CSS Modules** | Scoped, plain CSS | Manual design tokens |
| **Zero-runtime CSS-in-JS** (vanilla-extract, Panda CSS, Linaria) | Type-safe, no runtime cost | Build tooling |
| Runtime CSS-in-JS (styled-components, Emotion) | Dynamic styles | Runtime cost; friction with Server Components |
| Plain CSS with modern features | Cascade layers, container queries, nesting, `:has()` | Requires discipline |

### 11.2 Design system architecture

```text
Design tokens (colour, spacing, type, radius, motion) — single source (W3C Design Tokens format)
   → generated CSS variables / Tailwind theme / native platform tokens
   → primitive components (Button, Input, Dialog) built on accessible headless libraries
      (Radix UI, React Aria, Headless UI, Ark UI, Angular CDK)
   → patterns (forms, tables, empty states)
   → documented in Storybook with a11y + visual regression tests
```

```css
:root {
  --color-bg: #ffffff;
  --color-text: #1a1a1a;
  --color-primary: #0b57d0;
  --space-2: 0.5rem;
  --space-4: 1rem;
  --radius-md: 8px;
}
@media (prefers-color-scheme: dark) {
  :root { --color-bg: #121212; --color-text: #f2f2f2; --color-primary: #8ab4f8; }
}
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }
}
```

### 11.3 Responsive design

```text
- Mobile-first CSS; fluid typography with clamp(); container queries for component-level responsiveness
- Test at 320 px width, 200–400% zoom, landscape/portrait, and large screens
- Touch targets ≥ 24×24 CSS px (WCAG 2.2 minimum; 44–48 px recommended for primary actions)
```

---

## 12. Accessibility (WCAG 2.2)

Target **WCAG 2.2 Level AA**. Principles: Perceivable, Operable, Understandable, Robust.

### 12.1 Engineering checklist

| Area | Requirement |
|---|---|
| Semantics | Native elements; landmarks; one `<h1>`; logical heading order |
| Keyboard | Every interaction keyboard-operable; visible focus indicator; logical tab order; no keyboard traps |
| Focus management | Move focus to dialogs on open and back to the trigger on close; skip link to main content; route changes announce new page title |
| Focus not obscured (2.2) | Sticky headers/footers don't hide the focused element |
| Names and labels | Every control has an accessible name; icons-only buttons have `aria-label` |
| Images | Meaningful `alt`; decorative images `alt=""` |
| Colour & contrast | 4.5:1 text, 3:1 large text and UI components; never colour alone |
| Target size (2.2) | Minimum 24×24 CSS px or sufficient spacing |
| Dragging (2.2) | Single-pointer alternative for drag interactions |
| Forms | Labels, instructions, error identification and suggestions; accessible authentication (no cognitive tests without alternatives) (2.2) |
| Redundant entry (2.2) | Don't ask users to re-enter information already provided in the same process |
| Dynamic content | `aria-live` regions for async updates; status messages announced |
| Motion | Respect `prefers-reduced-motion`; no flashing > 3 times/second |
| Media | Captions, transcripts, audio descriptions where needed |
| Language | `<html lang="en">`; `lang` on passages in other languages (e.g. Hindi) |
| ARIA | First rule of ARIA: don't use ARIA if a native element works; follow the ARIA Authoring Practices Guide patterns |

### 12.2 Testing

```text
Automated in CI: eslint-plugin-jsx-a11y (or framework equivalent), axe-core in component and E2E tests (chapter 08 §20.3)
Manual each release: keyboard-only pass, screen reader pass (NVDA/JAWS on Windows, VoiceOver on macOS/iOS, TalkBack on Android),
                     200–400% zoom, high-contrast/forced-colors mode
Process: accessibility acceptance criteria in stories; a11y review in design; an accessibility statement published
```

---

## 13. Performance and Core Web Vitals

### 13.1 Core Web Vitals

| Metric | Measures | "Good" threshold (75th percentile of page loads) |
|---|---|---|
| **LCP** — Largest Contentful Paint | Loading | ≤ 2.5 s |
| **INP** — Interaction to Next Paint | Responsiveness (replaced FID as a Core Web Vital in March 2024) | ≤ 200 ms |
| **CLS** — Cumulative Layout Shift | Visual stability | ≤ 0.1 |

Supporting metrics: TTFB, FCP, Total Blocking Time (lab), long animation frames.

### 13.2 Field vs lab data

| Source | Tools |
|---|---|
| **Field (real users)** — decides priorities | CrUX (Chrome UX Report), PageSpeed Insights field section, RUM via `web-vitals` library → your analytics/observability |
| **Lab** — diagnose and prevent regressions | Lighthouse, Lighthouse CI, Chrome DevTools Performance panel, WebPageTest |

```ts
import { onLCP, onINP, onCLS } from 'web-vitals';
function send(metric: { name: string; value: number; id: string; rating: string }) {
  navigator.sendBeacon('/rum', JSON.stringify({ ...metric, page: location.pathname }));
}
onLCP(send); onINP(send); onCLS(send);
```

### 13.3 Fixes by metric

| Metric | Common causes | Fixes |
|---|---|---|
| **LCP** | Slow TTFB, render-blocking resources, large/lazy-loaded hero image, client-side rendering | CDN/edge caching, SSR/SSG, preload + `fetchpriority="high"` on LCP image, never lazy-load the LCP image, modern formats, reduce render-blocking CSS/JS |
| **INP** | Long tasks, heavy hydration, expensive event handlers, large DOM | Ship less JS, code split, yield to the main thread (`scheduler.yield()` where available, `setTimeout` chunking), web workers, `startTransition`, virtualise long lists, React Compiler to cut re-renders |
| **CLS** | Images/ads/embeds without dimensions, late-loading fonts, injected banners | Set `width`/`height` or `aspect-ratio`, reserve space, `font-display` + metric-matched fallbacks, avoid inserting content above existing content |

### 13.4 Performance budgets (enforce in CI)

```json
// Lighthouse CI assertions (excerpt) — lighthouserc.json
{
  "ci": {
    "assert": {
      "assertions": {
        "largest-contentful-paint": ["error", { "maxNumericValue": 2500 }],
        "cumulative-layout-shift": ["error", { "maxNumericValue": 0.1 }],
        "total-blocking-time": ["warn", { "maxNumericValue": 200 }],
        "resource-summary:script:size": ["error", { "maxNumericValue": 170000 }]
      }
    }
  }
}
```

| Budget (starting point, adapt per product) | Value |
|---|---|
| JS shipped on initial route (compressed) | ≤ ~150–200 KB |
| Initial CSS (compressed) | ≤ ~50 KB |
| LCP image | ≤ ~100–200 KB, responsive sizes |
| Third-party scripts | Each justified, loaded async/deferred, owned |

### 13.5 Bundle discipline

```text
- Analyse bundles every release (bundle analyser / Vite visualizer / Next.js bundle analysis)
- Route-level code splitting; dynamic import for heavy, rarely used components (charts, editors, maps)
- Prefer small libraries (date-fns/Temporal polyfill vs heavy date libs; lodash-es per-function imports)
- Third-party scripts (analytics, chat, tag managers) via async + consent gating; consider server-side tagging
```

---

## 14. Images, Fonts, and Media

### 14.1 Images

```html
<img
  src="/img/shoe-800.avif"
  srcset="/img/shoe-400.avif 400w, /img/shoe-800.avif 800w, /img/shoe-1600.avif 1600w"
  sizes="(max-width: 768px) 100vw, 50vw"
  width="800" height="800"
  alt="Blue running shoe, side view"
  fetchpriority="high"            <!-- only for the LCP image -->
  decoding="async" />
<!-- below-the-fold images: loading="lazy" -->
```

```text
- Formats: AVIF/WebP with fallbacks via <picture> where needed
- Responsive srcset/sizes; never ship a 3000 px image to a 360 px phone
- Always set dimensions (or aspect-ratio) to prevent CLS
- Use an image CDN or framework image component (next/image, @unpic, Astro Image) for resizing
```

### 14.2 Fonts

```text
- Self-host (privacy + performance) or use a fast font CDN; subset to needed scripts (e.g. Latin + Devanagari)
- WOFF2 only; preload only the critical font file(s)
- font-display: swap (or optional) + size-adjust/ascent-override fallbacks to reduce CLS
- Limit families/weights; consider variable fonts
- System font stacks for UI where brand allows
```

### 14.3 Video

`preload="none"` or `metadata` for non-hero videos, poster images, captions (`<track kind="captions">`), adaptive streaming (HLS/DASH) for long content, no autoplay with sound.

---

## 15. Internationalisation and Localisation

| Topic | Practice |
|---|---|
| Message catalogues | ICU MessageFormat (plurals, gender, select) via FormatJS/react-intl, i18next, Lingui, vue-i18n, Angular i18n, Paraglide |
| Formatting | `Intl.NumberFormat`, `Intl.DateTimeFormat`, `Intl.RelativeTimeFormat`, `Intl.PluralRules`, `Intl.ListFormat` — never hand-format |
| Currency | `new Intl.NumberFormat('en-IN', { style: 'currency', currency: 'INR' }).format(2499)` → "₹2,499.00" (Indian digit grouping handled) |
| Time zones | Store UTC; display in user's time zone (`Asia/Kolkata`) with `Intl` or the Temporal API/polyfill |
| Scripts & RTL | Support Devanagari, Tamil, etc. fonts; `dir="rtl"` and CSS logical properties (`margin-inline-start`) for Arabic/Urdu |
| Text expansion | Design for +30–40% string length |
| Locale routing | `/en-in/`, `/hi-in/` or domain/cookie-based; `hreflang` for SEO |
| Process | Externalise strings from day one; pseudo-localisation in CI to catch hard-coded text and truncation; translation management system (Crowdin, Lokalise, Phrase, Weblate) |

---

## 16. Frontend Security

| Threat | Mitigation |
|---|---|
| **XSS** | Framework auto-escaping; never `dangerouslySetInnerHTML` / `v-html` / `innerHTML` with untrusted data; DOMPurify for required rich HTML; strict CSP with nonces; Trusted Types where supported |
| **Clickjacking** | CSP `frame-ancestors 'none'` (or allow-list) |
| **CSRF** | SameSite cookies + CSRF tokens or Origin checks for cookie-authenticated mutations (frameworks' server actions often include origin checks — verify) |
| **Open redirects** | Validate `returnTo`/`next` params against an allow-list of relative paths |
| **Sensitive data exposure** | Never put secrets in client bundles or `NEXT_PUBLIC_*`/`VITE_*` env vars; review what's serialised into HTML |
| **Supply chain** | Lockfiles, SCA, minimal dependencies, Subresource Integrity (SRI) for third-party CDN scripts, review postinstall scripts |
| **Third-party scripts** | Inventory, CSP allow-listing, async loading, consent gating; PCI DSS v4.0.1 requires managing scripts on payment pages |
| **postMessage** | Check `event.origin`; specify target origin when sending |
| **Prototype pollution / unsafe deserialisation** | Validate parsed JSON with schemas |
| **Source maps** | Upload to error tracker privately; don't publish to production if they expose sensitive code/comments (policy decision) |

```text
CSP starting point (set by server with a fresh nonce per response):
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-{RANDOM}' 'strict-dynamic';
  style-src 'self' 'unsafe-inline'; img-src 'self' data: https://images.shopnow.example;
  connect-src 'self' https://api.shopnow.example; frame-ancestors 'none'; base-uri 'none'; object-src 'none';
  form-action 'self'; upgrade-insecure-requests
Roll out with Content-Security-Policy-Report-Only first, collect reports, then enforce.
```

Full security headers: chapter 09 §12.

---

## 17. Authentication in the Browser

| Pattern | Recommendation |
|---|---|
| First-party web app with own backend | **Session cookie** (`HttpOnly`, `Secure`, `SameSite=Lax`, `__Host-` prefix) — tokens stay server-side |
| SPA calling APIs on other domains | **BFF pattern**: BFF does the OIDC flow (Authorization Code + PKCE) and holds tokens; browser uses session cookie |
| Pure SPA without BFF (last resort) | Authorization Code + PKCE; access tokens in memory only, short-lived; refresh via secure mechanisms; never `localStorage` for long-lived tokens |
| Passwordless | Passkeys/WebAuthn (`navigator.credentials.create/get`) with conditional UI (autofill) |

```text
🔴 Logout clears server session and client caches (query cache, service worker caches with user data)
🔴 Route guards in the UI are UX only — APIs enforce authorisation
🟠 Handle session expiry gracefully: preserve user work, re-authenticate, resume
```

Details: chapter 09 §8–9.

---

## 18. Testing the Frontend

| Level | Tools | What to test |
|---|---|---|
| Static | TypeScript, ESLint (incl. a11y, React hooks rules), Stylelint | Types, bug patterns, a11y lint |
| Unit | Vitest/Jest | Pure utilities (money, dates, formatting), hooks, reducers |
| Component | Testing Library (+ Vitest/Jest), Storybook interaction tests, Playwright/Cypress component testing | Behaviour from the user's perspective: render, interact, assert visible output |
| Integration | Testing Library + MSW (mock network) | Feature flows within the app |
| E2E | Playwright (preferred), Cypress | Critical journeys across real backend/staging |
| Visual regression | Playwright screenshots, Chromatic, Percy | Unintended UI changes |
| Accessibility | axe in component/E2E tests + manual | WCAG 2.2 AA |
| Performance | Lighthouse CI, bundle size checks | Budgets |

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('shows alternative methods after a declined payment', async () => {
  server.use(declinedPaymentHandler);                         // MSW handler
  render(<Checkout orderId="ord_7f3kq2m9x1" />);
  await userEvent.click(screen.getByRole('button', { name: /pay/i }));
  expect(await screen.findByRole('alert')).toHaveTextContent(/declined/i);
  expect(screen.getByRole('button', { name: /pay with upi/i })).toBeVisible();
});
```

```text
Query priority (Testing Library): getByRole > getByLabelText > getByPlaceholderText > getByText > getByTestId
→ tests that use roles double as accessibility checks.
```

Full strategy: chapter 08.

---

## 19. Build Tooling

| Tool | Role |
|---|---|
| **Vite** | Dev server + production builds for most non-Next.js frameworks (React, Vue, Svelte, Solid) |
| **Turbopack** | Default bundler in Next.js 16 |
| Rspack / Rsbuild | Webpack-compatible, Rust-based, fast |
| webpack | Legacy/complex setups |
| esbuild / SWC / Oxc | Fast transpilers/minifiers used inside the above |
| **Package manager** | pnpm (recommended for monorepos), npm, Yarn, Bun — see **37** |
| Monorepo | Nx, Turborepo (chapter 07 §4.3) |
| Lint/format | ESLint + Prettier, or Biome (chapter 06 §8) |

```text
🔴 Reproducible builds: lockfile + `npm ci` / `pnpm install --frozen-lockfile`
🔴 Environment-specific config at runtime/deploy time where possible; build-time public env vars contain no secrets
🟠 Source maps uploaded to error tracking per release
🟠 Hashed asset filenames + long-lived immutable caching; HTML short-cached
```

---

## 20. Observability and Error Tracking

| Signal | Tooling |
|---|---|
| JS errors and unhandled rejections | Sentry, Datadog RUM, New Relic Browser, Elastic RUM, Grafana Faro, Bugsnag |
| Core Web Vitals (field) | `web-vitals` → RUM backend; CrUX |
| Distributed tracing | Browser OpenTelemetry instrumentation; propagate `traceparent` to APIs (CORS must allow the header) |
| Product analytics | Privacy-respecting analytics with consent (DPDP/GDPR) |
| Session replay | Only with consent and strict PII masking |

```text
🔴 Release tagging: every event carries app version/commit SHA
🔴 PII scrubbing in error payloads and breadcrumbs
🟠 Error budgets/SLOs for key frontend journeys (e.g. checkout success rate, JS error rate per session)
🟠 Alert on spikes in JS errors after deploys (correlate with release markers)
```

---

## 21. SEO and Social Sharing

```text
- Server-render or pre-render indexable pages; unique <title> and meta description per page
- Canonical URLs; hreflang for locales; XML sitemap; robots.txt
- Structured data (JSON-LD, schema.org): Product, Offer, BreadcrumbList, Organization, FAQ where appropriate
- Open Graph and Twitter/X card meta tags with images
- Clean, stable URLs; 301 redirects for moved content; real 404 status for missing pages
- Core Web Vitals and mobile-friendliness affect page experience
- Don't block CSS/JS needed for rendering in robots.txt
```

```html
<script type="application/ld+json">
{ "@context": "https://schema.org", "@type": "Product", "name": "Blue Running Shoe",
  "image": "https://shopnow.example/img/shoe-800.avif",
  "offers": { "@type": "Offer", "priceCurrency": "INR", "price": "2499.00", "availability": "https://schema.org/InStock" } }
</script>
```

---

## 22. PWAs, Offline, and Service Workers

| Capability | Notes |
|---|---|
| Web App Manifest | Name, icons, theme colour, display mode → installable |
| Service worker caching | Workbox strategies: cache-first for static assets, stale-while-revalidate for content, network-first for APIs |
| Offline | Offline fallback page; queue mutations with Background Sync where supported; IndexedDB for local data |
| Push notifications | Web Push (supported on iOS for installed web apps since iOS 16.4); always ask permission in context |

```text
🔴 Version caches and clean old ones on activate; never cache authenticated API responses in shared caches without user scoping
🔴 Clear user-specific caches on logout
🟠 Test update flows: new service worker waiting → prompt user to reload
```

---

## 23. Micro-Frontends

Split a large frontend into independently developed and deployed parts owned by different teams.

| Approach | Notes |
|---|---|
| Route-based split (separate apps per path behind a reverse proxy) | Simplest; full page loads between areas |
| Build-time composition (packages) | Shared release train; simpler runtime |
| Runtime composition (Module Federation, import maps, Web Components) | Independent deploys; complexity in shared deps and versioning |
| Server-side composition (edge includes, fragments) | Good performance; infra complexity |

```text
Adopt only when team scaling demands it (many teams on one UI). Costs: duplicated dependencies, inconsistent UX,
cross-app state and routing complexity, harder performance budgets. Mitigate with a shared design system,
shared dependency versions, contracts between fragments, and a platform team.
```

---

## 24. Deployment and Delivery

| Model | Platforms |
|---|---|
| Static + CDN | Cloudflare Pages, Netlify, Vercel, S3/CloudFront, Azure Static Web Apps, Firebase Hosting, Nginx on a VPS (**39**) |
| SSR/edge hosting | Vercel, Netlify, Cloudflare Workers, AWS Amplify/Lambda@Edge/CloudFront Functions, containers (Cloud Run, ECS, Kubernetes), Node server on a VPS with PM2/systemd behind Nginx |
| Preview deployments | Per-PR environments for review and E2E (chapter 08 §8) |

```text
🔴 Immutable, versioned deployments; instant rollback
🔴 Cache headers: hashed assets → Cache-Control: public, max-age=31536000, immutable; HTML → no-cache or short max-age
🔴 Security headers and CSP at the edge or server (chapter 09 §12)
🟠 Feature flags for risky UI changes; gradual rollout
🟠 Handle version skew: old clients calling new APIs (and new chunks after deploy) — keep old assets available
    for a period; detect chunk-load errors and prompt refresh
```

---

## 25. Dependency and Upgrade Management

```text
🔴 Automated update PRs (Renovate/Dependabot) grouped sensibly (framework group, lint group, types group)
🔴 Security advisories subscribed for your framework (Next.js/React, Angular, Vue/Nuxt, SvelteKit)
🟠 Upgrade majors within ~one cycle; use official codemods (e.g. Next.js, React, Angular `ng update`)
🟠 Track deprecations (React, browser APIs) with lint rules
🟠 Keep Node.js on an Active/Maintenance LTS line for build and SSR (chapter 37/41 for installs)
```

---

## 26. Checklists

### New frontend project
- [ ] Framework and rendering strategy chosen with ADR
- [ ] TypeScript strict; ESLint (incl. a11y) + Prettier/Biome; browserslist defined
- [ ] Feature-based structure with enforced boundaries
- [ ] API types generated from contracts; MSW mocks
- [ ] Design tokens + accessible component library; Storybook
- [ ] Auth via session cookies/BFF; no tokens in localStorage
- [ ] CSP (report-only → enforce) and security headers
- [ ] i18n wiring and `Intl` formatting from day one
- [ ] RUM (web-vitals) + error tracking with release tags and PII scrubbing
- [ ] Lighthouse CI budgets + bundle size checks in CI
- [ ] Playwright E2E for critical journeys; axe checks
- [ ] Preview deployments per PR; immutable deploys with rollback

### Every UI PR
- [ ] Keyboard and screen-reader friendly (roles, labels, focus)
- [ ] Loading/empty/error states handled
- [ ] No new heavy dependencies without justification; bundle impact checked
- [ ] Images sized, responsive, lazy-loaded below the fold
- [ ] Strings externalised for translation
- [ ] Tests at the right level (component + a11y; E2E only if critical journey changed)
- [ ] No secrets or sensitive data exposed to the client

### Release
- [ ] Core Web Vitals (field) within thresholds for key pages
- [ ] No new a11y violations; manual screen-reader pass on changed flows
- [ ] Error rate monitored after deploy; rollback ready
- [ ] Framework/security advisories reviewed

---

## 27. References

### Platform and standards
- MDN Web Docs: https://developer.mozilla.org/
- web.dev (performance, Core Web Vitals, Baseline): https://web.dev/
- Core Web Vitals: https://web.dev/articles/vitals
- INP: https://web.dev/articles/inp
- Baseline: https://web.dev/baseline
- WCAG 2.2: https://www.w3.org/TR/WCAG22/
- WAI-ARIA Authoring Practices Guide: https://www.w3.org/WAI/ARIA/apg/
- HTML Living Standard: https://html.spec.whatwg.org/

### Frameworks and tools
- React: https://react.dev/ (React 19.2 blog, React Compiler v1)
- Next.js: https://nextjs.org/docs · Next.js 16 release: https://nextjs.org/blog
- Angular: https://angular.dev/ · Vue: https://vuejs.org/ · Nuxt: https://nuxt.com/ · Svelte/SvelteKit: https://svelte.dev/ · Astro: https://astro.build/
- React Router: https://reactrouter.com/ · TanStack (Query, Router, Start): https://tanstack.com/
- Vite: https://vite.dev/ · Tailwind CSS: https://tailwindcss.com/
- Testing Library: https://testing-library.com/ · Playwright: https://playwright.dev/ · MSW: https://mswjs.io/ · Storybook: https://storybook.js.org/
- Lighthouse CI: https://github.com/GoogleChrome/lighthouse-ci · web-vitals: https://github.com/GoogleChrome/web-vitals
- Workbox: https://developer.chrome.com/docs/workbox/
- FormatJS: https://formatjs.github.io/ · i18next: https://www.i18next.com/

### Security
- OWASP Cheat Sheets — XSS Prevention, CSP, DOM-based XSS: https://cheatsheetseries.owasp.org/
- CSP Evaluator: https://csp-evaluator.withgoogle.com/
- OAuth 2.0 for Browser-Based Apps (IETF draft, BFF pattern): https://datatracker.ietf.org/doc/draft-ietf-oauth-browser-based-apps/

---

**Previous:** [11 — API & Integration](./11-api-and-integration.md) · **Next:** [13 — Backend Engineering](./13-backend-engineering.md)