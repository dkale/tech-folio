# Component-Based Micro Frontends

## Table of Contents

- [Core Concept at Runtime](#core-concept-at-runtime)
- [Real Examples](#real-examples)
- [Frameworks and Runtime Patterns](#frameworks-and-runtime-patterns)
- [How Runtime Composition Works](#how-runtime-composition-works)
- [Key Engineering Contracts](#key-engineering-contracts)
- [Is Component-Based Micro Frontends Different from Micro Frontends?](#is-component-based-micro-frontends-different-from-micro-frontends)

---

## Core Concept at Runtime

At runtime, the UI is assembled from independent, self-contained frontend components (micro frontends) that are developed and deployed separately, then composed into one user experience by a host or container app.

In practice:

1. Each team owns a slice of UI and its logic.
2. The host loads those slices dynamically at runtime, not as one large prebuilt bundle.
3. Shared contracts (APIs, events, and design system rules) keep slices interoperable without tight coupling.

Core idea: independent deployability plus runtime composition.

---

## Real Examples

### 1) E-commerce storefront

- Team A owns Header and Search.
- Team B owns Product Catalog.
- Team C owns Cart.
- Team D owns Checkout.
- The shell loads each part from separate deployment URLs at runtime.
- If Cart ships a bug fix, only Cart is redeployed.

### 2) Banking portal

- Accounts, Transfers, Cards, and Investments are separate micro frontends.
- Users authenticate once through the shell.
- Modules are loaded on demand by route.
- A large Investments bundle does not impact users who only check balances.

### 3) Enterprise dashboard

- HR, Payroll, IT Tickets, and Analytics are owned by different domain teams.
- Teams can use different frameworks if integration contracts are respected.
- The shell standardizes navigation, auth, and design tokens.

---

## Frameworks and Runtime Patterns

### 1) Webpack Module Federation (React, Angular, Vue)

- Most common modern implementation pattern.
- A host dynamically imports remote modules exposed by other apps.
- Works well when teams want shared dependencies and less duplication.

Example stack:

- Host: React
- Remote 1: React Product List
- Remote 2: Angular Admin Tools
- Composition: runtime loading via remote manifests

### 2) single-spa

- Framework-agnostic orchestrator for micro frontends.
- Mounts and unmounts apps based on route or activity rules.
- Useful when multiple frameworks must coexist over time.

Example:

- Navbar in Vue
- Shopping flow in React
- Legacy reports in AngularJS

### 3) qiankun

- Popular when strong sandboxing and orchestration are needed.
- Built on single-spa ideas with extra isolation capabilities.
- Useful for large multi-team portals with CSS and JS isolation concerns.

### 4) Web Components

- Uses native browser custom elements.
- Each micro frontend ships as reusable custom elements.
- Works well across framework boundaries.

Example components:

- `<account-summary>`
- `<recent-transactions>`
- `<loan-offers>`

### 5) iframe-based micro frontends

- Older but still valid for strict isolation and security boundaries.
- Strongest isolation, but weaker UX integration and cross-app communication.
- Common for legacy systems or partner-hosted experiences.

---

## How Runtime Composition Works

1. Shell bootstraps and loads global auth, layout, navigation, and feature flags.
2. Route logic decides which micro frontend is required.
3. The shell fetches remote entry manifests and JavaScript chunks.
4. The remote app is mounted into a shell-owned DOM slot.
5. Apps communicate through events, shared contracts, or API responses.
6. Each micro frontend can be released independently if contracts stay stable.

---

## Key Engineering Contracts

1. UI contract: design tokens, accessibility standards, and shared component rules.
2. Integration contract: route ownership, mount points, and lifecycle APIs.
3. Data contract: API schemas, versioning strategy, and error formats.
4. Runtime contract: shared dependency policy, caching, and observability hooks.

Rule of thumb:

If teams need independent release velocity and strong domain ownership, runtime micro frontends are a strong fit. If one small team owns a simple app, the runtime complexity may outweigh benefits.

---

## Is Component-Based Micro Frontends Different from Micro Frontends?

Short answer: usually not fundamentally.

- Micro frontends is the umbrella architecture: split the frontend into independently owned and deployable parts.
- Component-based micro frontends is one implementation style inside that umbrella, where integration happens at component level rather than only at route or page level.

Relationship:

1. Micro frontends = broad architecture pattern.
2. Component-based micro frontends = one implementation approach.

Practical difference:

- Route-based micro frontends: teams own full pages or routes.
- Component-based micro frontends: teams own smaller fragments (for example header widgets, cart panels, recommendation cards) that are composed at runtime.
