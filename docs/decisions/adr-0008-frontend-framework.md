# ADR-0008: Frontend Framework: Next.js 14+ (App Router, React 18)

**Status**: Accepted
**Date**: 2026-10-06
**Deciders**: Rishi Sakhija
**Technical Story**: Frontend technology selection

## Context

We need a modern React framework for the search UI, briefing feed, entity pages, research workspace, and settings. Requirements:

- Server-side rendering (SEO, initial load performance)
- React Server Components (data fetching in components)
- TypeScript first-class
- Good developer experience (hot reload, type safety)
- Vercel deploy (or self-hosted Docker)
- Component library compatible (Radix UI, shadcn/ui)
- WebSocket support (real-time research progress, briefing updates)

## Decision

**Choose Next.js 14+ with App Router**.

## Alternatives Considered

| Alternative | Pros | Cons |
|-------------|------|------|
| **Remix** | Web standards; nested routes; great DX | Smaller ecosystem; no RSC; Shopify-owned |
| **SvelteKit** | Fast; small bundles; great DX | Smaller talent pool; different paradigm |
| **Astro** | Island architecture; fast static | Not ideal for highly interactive app |
| **Vite + React (SPA)** | Simple; full control | No SSR; SEO poor; manual data fetching |
| **TanStack Start** | Modern; type-safe routing | Beta; less mature; smaller ecosystem |

## Rationale

- **App Router + RSC** = data fetching in components, no prop drilling, streaming SSR
- **Vercel** = zero-config deploy; edge functions; preview deployments
- **Ecosystem** = largest React meta-framework; shadcn/ui, Radix, Tailwind all work seamlessly
- **TypeScript** = first-class; route types; server actions typed
- **WebSockets** = supported via custom server or Pusher/Ably integration
- **Team familiarity** = React/Next.js widely known

## Consequences

### Positive
- Fast initial loads (streaming SSR)
- SEO-friendly (search pages, entity pages indexed)
- Great DX (Turbopack, hot reload, type-safe routes)
- Incremental adoption (can mix client/server components)

### Negative
- **App Router complexity**: Server/Client boundary mental model
- **Bundle size**: Next.js adds overhead vs. Vite SPA
- **Vendor tie-in**: Vercel features (ISR, Edge) not portable
- **Learning curve**: RSC patterns new to some

### Risks
- **RSC hydration issues**: Test thoroughly; use `use client` judiciously
- **Mitigation**: Start with client components for interactive parts; migrate to RSC where beneficial

## Implementation Notes

- **Styling**: Tailwind CSS + shadcn/ui (Radix primitives)
- **State**: React Query (TanStack Query) for server state; Zustand for client state
- **Forms**: React Hook Form + Zod
- **Charts**: Recharts / Tremor
- **Real-time**: WebSocket client (native WS or Socket.io client) for research progress
- **Auth**: NextAuth.js (Auth.js) for OAuth + JWT
- **Testing**: Vitest (unit), Playwright (E2E)

## Related ADRs

- ADR-0001: Search Architecture (UI consumes Search API)
- ADR-0009: Primary Database (User service)