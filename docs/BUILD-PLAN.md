# Build Plan & OSS Issue Backlog

This document is the **implementation source of truth** for contributors building the Trustless Work Payment Widget template. Use it to understand architecture, scope, and to **create GitHub issues** one-to-one from the sections below.

**Related docs**

| Document | Purpose |
| --- | --- |
| [`PRODUCT-BRIEF.md`](./PRODUCT-BRIEF.md) | Product concept, widget API, UX, MVP scope, acceptance criteria |
| [`.cursor/rules/FOLDER_LAYOUT.mdc`](../.cursor/rules/FOLDER_LAYOUT.mdc) | Vertical slice architecture (`src/features/`) |
| [Trustless Work Blocks docs](https://docs.trustlesswork.com/trustless-work/escrow-blocks-sdk/getting-started) | Escrow Blocks SDK |
| [`.agents/skills/trustless-work/`](../.agents/skills/trustless-work/) | Local skill: SDK, Blocks, XDR signing, roles |

---

## How to use this document

1. **Creating GitHub issues?** Follow [Issue creation order](#issue-creation-order-for-agents--maintainers) below — **do not create issues in random or numeric-only order**.
2. **Implementing work?** Only pick issues whose **Blocked by** dependencies are already closed/merged.
3. **Copy the issue template** into GitHub Issues (title, labels, body, acceptance criteria).
4. **Follow architecture rules** — routing stays thin; domain code lives under `src/features/`.
5. **Cross-check MVP scope** in `PRODUCT-BRIEF.md` §8 and §24 before adding features.

### Suggested GitHub labels

| Label | Use for |
| --- | --- |
| `epic:setup` | Infrastructure, providers, deps |
| `epic:widget-ui` | Custom payment widget shell components |
| `epic:mock` | Mock mode without API |
| `epic:demo` | Demo app pages and views |
| `epic:blocks` | `@trustless-work/blocks` integration |
| `epic:services` | Mapping, utils, payload builders |
| `epic:docs` | Documentation |
| `priority:must-have` | MVP blockers |
| `priority:should-have` | Post-MVP polish within v1 |
| `good first issue` | Small, well-scoped, low deps |
| `help wanted` | Ready for external contributors |

---

## Architecture decisions

### 1. Vertical slices (`src/features/`)

Code is organized by **product domain**, not by technical layer at the repo root.

| Location | Responsibility |
| --- | --- |
| `src/app/` | Thin route files — import views, pass params only |
| `src/features/payment-widget/` | Widget domain: components, hooks, services, schemas, utils |
| `src/features/demo/` | Demo app views and demo-only constants |
| `src/components/ui/` | shadcn/ui primitives (global design system) |
| `src/components/escrows/` | Escrow Blocks installed by CLI (do not duplicate) |
| `src/providers/` | App-wide providers (Wallet) |
| `src/types/` | Cross-feature TypeScript contracts |

**Dependency rule:** `features/demo` may import `features/payment-widget`. The payment widget must **not** import demo internals.

### 2. Compose Escrow Blocks — do not reimplement

`@trustless-work/blocks` provides production-ready UI for escrow operations (forms, dialogs, tables, wallet connect). The Payment Widget is a **checkout-style shell** that composes blocks plus custom pieces (timeline, fees, modes, mock).

| MVP step | Escrow Block / SDK | Custom in this repo |
| --- | --- | --- |
| Create escrow | `InitializeEscrowForm` | Payload mapping from widget props |
| Deposit funds | `useFundEscrow` (SDK hook) | **Custom UI** — no dedicated block listed |
| Mark delivered | `ChangeMilestoneStatus` | Evidence form (should-have) |
| Approve | `ApproveMilestone` | Role gating in orchestrator |
| Release | `ReleaseEscrow` | Role gating in orchestrator |
| List / select escrow | `EscrowsByRole`, `EscrowsBySigner` | `setSelectedEscrow` wiring |
| Connect wallet | `WalletButton` | `WalletProvider` (Stellar Wallets Kit) |

**Blocks context pattern:** call `setSelectedEscrow(escrow)` from `@trustless-work/blocks` before operation components in `existing` mode so `ApproveMilestone`, `ReleaseEscrow`, etc. read `contractId` and roles automatically.

### 3. Provider order (non-negotiable)

```
QueryClientProvider          ← @tanstack/react-query
  └── TrustlessWorkConfig    ← @trustless-work/escrow
        └── WalletProvider   ← @creit.tech/stellar-wallets-kit
              └── {children}
```

Create `QueryClient` at **module scope**, not inside a React component. All provider wrappers must be Client Components (`"use client"`).

### 4. XDR signing pattern

Every write operation: **API/hook → unsigned XDR → sign with correct role wallet → submit**. See `.agents/skills/trustless-work/SKILL.md`.

### 5. Mock mode vs real mode

| Mode | Trigger | Behavior |
| --- | --- | --- |
| **Mock** | `NEXT_PUBLIC_USE_MOCK=true` or widget prop | Local state machine; blocks hidden; no API key |
| **Real** | Default when mock flag off + API key set | Blocks + SDK hooks + wallet signing |

Mock mode is a **must-have** for OSS onboarding (see `PRODUCT-BRIEF.md` §8).

---

## Tech stack & dependencies

### Core (already in repo)

- Next.js 16 (App Router) + React 19 + TypeScript (`strict: true`)
- Tailwind CSS v4

### Required installs

Install peer dependencies explicitly. Blocks does **not** bundle them.

```bash
pnpm add @trustless-work/blocks @trustless-work/escrow \
  @tanstack/react-query @tanstack/react-query-devtools @tanstack/react-table \
  @creit.tech/stellar-wallets-kit \
  react-hook-form @hookform/resolvers zod axios sonner
```

### What each dependency is for

| Package | Used by | Notes |
| --- | --- | --- |
| `@trustless-work/blocks` | Escrow forms, dialogs, tables, `WalletButton`, `useEscrowContext` | Primary UI for escrow ops |
| `@trustless-work/escrow` | `TrustlessWorkConfig`, all write/read hooks | Required peer of blocks |
| `@tanstack/react-query` | Data fetching, cache, mutations | Provider required outermost |
| `@tanstack/react-table` | `EscrowsByRole`, `EscrowsBySigner` | Peer dep of blocks |
| `@tanstack/react-query-devtools` | Local dev | Dev only |
| `react-hook-form` + `@hookform/resolvers` + `zod` | Block forms + custom `EvidenceForm` | Blocks use these internally |
| `axios` | SDK/blocks HTTP layer | Peer dep |
| `@creit.tech/stellar-wallets-kit` | Wallet connect + XDR signing | You implement `WalletProvider` |
| `sonner` | Toasts for success/error | Project convention (shadcn) |
| **shadcn/ui** | All UI primitives | Installed via `npx trustless-work init` |

### CLI bootstrap (recommended)

```bash
npx trustless-work init          # shadcn + peer deps + .twblocks.json
npx trustless-work add wallet-kit
npx trustless-work escrows       # copies blocks → src/components/escrows/
```

### Environment variables

```env
NEXT_PUBLIC_API_KEY=             # Trustless Work API key (real mode)
NEXT_PUBLIC_USE_MAINNET=false    # testnet default
NEXT_PUBLIC_USE_MOCK=true        # mock mode for local demo without keys
# Optional role/asset config for demos:
NEXT_PUBLIC_USDC_ISSUER_ADDRESS= # G… issuer, NOT C… contract
NEXT_PUBLIC_PLATFORM_ADDRESS=
```

See `.env.local.example` in repo root.

---

## Target repository structure

```
src/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   └── demo/
│       ├── product/[slug]/page.tsx
│       └── orders/[id]/
│           ├── buyer/page.tsx
│           ├── seller/page.tsx
│           └── admin/page.tsx
│
├── components/
│   ├── ui/                    # shadcn
│   ├── Providers.tsx          # QueryClient → TrustlessWorkConfig → Wallet
│   └── escrows/               # CLI-installed blocks
│
├── providers/
│   └── WalletProvider.tsx
│
├── types/
│   ├── payment-widget.ts
│   ├── escrow.ts
│   ├── roles.ts
│   └── fees.ts
│
└── features/
    ├── payment-widget/
    │   ├── components/
    │   │   ├── TrustlessPaymentWidget.tsx
    │   │   ├── PaymentWidgetCard.tsx
    │   │   ├── PaymentStatusTracker.tsx
    │   │   ├── FeeBreakdown.tsx
    │   │   ├── RoleSummary.tsx
    │   │   ├── FundEscrowAction.tsx
    │   │   ├── ErrorState.tsx
    │   │   └── SuccessReceipt.tsx
    │   ├── hooks/
    │   │   ├── usePaymentWidget.ts
    │   │   ├── usePaymentEscrow.ts
    │   │   └── useMockPaymentEscrow.ts
    │   ├── services/
    │   │   ├── mapEscrowStatus.ts
    │   │   └── buildEscrowPayload.ts
    │   ├── schemas/
    │   │   └── evidence.schema.ts
    │   ├── utils/
    │   │   ├── calculateFees.ts
    │   │   ├── formatCurrency.ts
    │   │   └── shortenAddress.ts
    │   └── constants/
    │       ├── status-labels.ts
    │       └── timeline-steps.ts
    │
    └── demo/
        ├── views/
        │   ├── ProductDemoView.tsx
        │   ├── BuyerOrderView.tsx
        │   ├── SellerOrderView.tsx
        │   └── AdminOrderView.tsx
        ├── components/
        │   └── OrderSummary.tsx
        └── constants/
            └── mock-order.ts
```

---

## Payment widget state machine (reference)

Internal states (`PaymentEscrowStatus`) — full list in `PRODUCT-BRIEF.md` §11.

```
idle → creating_escrow → awaiting_deposit → funded → awaiting_delivery
  → delivered → awaiting_approval → approved → ready_to_release
  → releasing → released
```

Additional UI states: `cancelled`, `disputed`, `expired`, `error`.

Service `mapEscrowStatus.ts` translates Trustless Work indexer/API data (milestones, flags) into these widget states.

---

## Default role mapping (v1)

| Trustless Work role | Widget meaning | Default actor |
| --- | --- | --- |
| Funder | Payer / buyer | Buyer |
| Receiver | Seller / provider | Seller |
| Service Provider | Milestone marker (delivered) | Seller |
| Approver | Approves delivery | Buyer |
| Release Signer | Triggers release | Buyer (platform optional) |
| Platform Address | Platform fee recipient | Platform |

Single-release escrow, **one milestone** for MVP.

---

## Issue dependency graph (implementation)

High-level flow — **not** the order for creating GitHub issues (see next section).

```
Epic 1 (Setup)
  ├── #2 Blocks CLI & deps
  ├── #3 Providers
  ├── #4 WalletProvider
  ├── #5 Folder structure
  └── #6 Types
        │
        ▼
Epic 2 (Widget UI shell) ──► Epic 3 (Utils/Services)
        │                            │
        └────────────┬───────────────┘
                     ▼
              Epic 4 (Mock mode)
                     │
         ┌───────────┴───────────┐
         ▼                       ▼
   Epic 5 (Demo app)      Epic 6 (Blocks integration)
         │                       │
         └───────────┬───────────┘
                     ▼
         Epic 7 (Orchestration hooks)
                     │
                     ▼
              Epic 8 (Documentation)
                     │
                     ▼
         Epic 9 (Should-have polish)
```

---

## Issue creation order (for agents & maintainers)

> **Read this before creating GitHub issues.**  
> Issue numbers in this doc (`#1`–`#43`) are **logical IDs**. When you create issues on GitHub, each new issue gets a **real** issue number. Always link **GitHub issue numbers** in `Blocked by` / `Blocks` — not BUILD-PLAN IDs alone.

### Rules when creating issues

1. **Follow the phase sequence below** — create issues phase by phase, top to bottom. Within a phase, create issues **left to right** (order in the row matters when listed sequentially).
2. **Never skip a phase** — later phases reference blockers from earlier phases. If phase N is not created yet, phase N+1 issues will have broken dependency links.
3. **Do not create all 43 issues in numeric order `#1 → #43`** — some lower IDs depend on higher IDs (e.g. `#9` is blocked by `#13`). Use the **phase table**, not raw issue number sorting.
4. **Always fill `Blocked by`** in the issue body using **existing GitHub issue numbers** from prior phases.
5. **One issue per BUILD-PLAN entry** — do not merge or split without maintainer approval.
6. **Epic 9 (#40–#43)** — create **last**, after MVP issues exist, so contributors do not pick polish work before core flow.
7. **Docs that can start early:** `#37` (role mapping) has no blockers — include it in Phase 5, not at the end.

### Phase sequence (creation order)

Create issues in exactly these phases. Finish a phase before starting the next.

| Phase | Create these BUILD-PLAN issues (in this order) | Why this batch |
| --- | --- | --- |
| **P1** | `#1` | Base template; everything else assumes repo runs |
| **P2** | `#2`, `#5` | Toolchain + folder scaffold (both only need `#1`) |
| **P3** | `#4`, `#6` | Wallet needs deps from `#2`; types need folders from `#5` |
| **P4** | `#3` | Providers need `#2` + `#4` GitHub issues to exist for linking |
| **P5** | `#13`, `#14`, `#15`, `#8`, `#11`, `#17`, `#19`, `#37` | Utils/services + early UI + mock data + landing + role docs (all depend only on P1–P4) |
| **P6** | `#7`, `#9`, `#16`, `#24` | `#9` needs `#13`; `#16` needs `#14`; `#24` needs `#2` |
| **P7** | `#10`, `#12` | `#10` needs `#14`; `#12` needs `#7`, `#8`, `#9` |
| **P8** | `#18` | Mock wired into widget — needs `#12` + `#16` |
| **P9** | `#20`, `#21`, `#22`, `#23` | Demo pages — need `#18` (+ `#17` for `#20`) |
| **P10** | `#25`, `#29`, `#31`, `#33` | Blocks create, fund, wallet, fetch hook (parallel once P7–P8 done) |
| **P11** | `#30` | Escrow context — needs `#33` + `#24` + `#14` |
| **P12** | `#26`, `#27`, `#28` | Deliver, approve, release blocks — need `#30` |
| **P13** | `#32` | Orchestrator hook — needs mock + all block integrations |
| **P14** | `#34`, `#35` | Callbacks + role visibility — need `#32` |
| **P15** | `#36`, `#38`, `#39` | Component API, demo script, README |
| **P16** | `#40`, `#41`, `#42`, `#43` | Should-have polish (optional for MVP) |

### Compact creation checklist (copy for agents)

```
P1:  #1
P2:  #2 → #5
P3:  #4 → #6
P4:  #3
P5:  #13 → #14 → #15 → #8 → #11 → #17 → #19 → #37
P6:  #7 → #9 → #16 → #24
P7:  #10 → #12
P8:  #18
P9:  #20 → #21 → #22 → #23
P10: #25 → #29 → #31 → #33
P11: #30
P12: #26 → #27 → #28
P13: #32
P14: #34 → #35
P15: #36 → #38 → #39
P16: #40 → #41 → #42 → #43
```

### Parallel creation within a phase

Issues in the **same phase row** may be created in the same session **if** all prior phases are already on GitHub. Within a phase:

| Phase | Safe to create in parallel |
| --- | --- |
| P2 | `#2` and `#5` — independent |
| P3 | `#4` and `#6` — independent |
| P5 | All eight issues — independent of each other |
| P6 | All four issues — independent of each other |
| P9 | All four demo pages — independent of each other |
| P10 | All four issues — independent of each other |
| P12 | `#26`, `#27`, `#28` — independent of each other |
| P14 | `#34` and `#35` — independent |
| P16 | `#40`–`43` — independent (low priority) |

**Never parallelize across phases** — e.g. do not create `#25` (P10) before `#12` (P7) exists on GitHub.

### Implementation order vs creation order

| Concern | Order to follow |
| --- | --- |
| **Creating GitHub issues** | Phase sequence P1→P16 above |
| **Assigning / implementing work** | Same dependency graph, but contributors may work **in parallel within a milestone** once blockers are **merged** (e.g. `#7` and `#13` after `#6` is done) |
| **First demoable MVP** | Complete through **P9** (mock flow + demo pages) before requiring testnet (P10+) |

### Mapping phases to milestones

| Milestone | Create phases | BUILD-PLAN issues |
| --- | --- | --- |
| M1 — Foundation | P1–P4 | `#1`–`#6` |
| M2 — Widget shell + mock | P5–P8 | `#7`–`#18` |
| M3 — Demo app | P9 | `#19`–`#23` |
| M4 — Blocks real mode | P10–P12 | `#24`–`#31` |
| M5 — Orchestration | P13–P14 | `#32`–`#35` |
| M6 — Docs & polish | P15–P16 | `#36`–`#43` |

### Agent prompt snippet

When instructing another agent to create issues, include:

> Create GitHub issues from `docs/BUILD-PLAN.md` **strictly following the Phase sequence (P1→P16)**. For each issue, set `Blocked by` to **GitHub issue numbers** from earlier phases only. Do not create Epic 6+ issues before Epic 1–4 issues exist on GitHub. Do not create `#9` before `#13`. Finish P1–P9 before P10 if the goal is an MVP mock demo first.

---

# Epic 1 — Project Setup & Infrastructure

> **Goal:** Runnable app with Blocks toolchain, providers, folder layout, and shared types.  
> **Labels:** `epic:setup`, `priority:must-have`

---

## Issue #1 — Verify Next.js base template

**Status:** Partially done (scaffold exists; demo routes missing)

**Description**  
Confirm the Next.js 16 + TypeScript + Tailwind v4 scaffold runs locally. Replace the default create-next-app homepage with a minimal landing that links to demo routes (placeholders OK).

**Tasks**

- [ ] `pnpm dev` runs without errors
- [ ] `pnpm build` passes
- [ ] Update `layout.tsx` metadata (title, description)
- [ ] Add placeholder routes under `src/app/demo/` (can render "Coming soon")

**Acceptance criteria**

- App runs at `http://localhost:3000`
- Routes exist (placeholders OK):
  - `/demo/product/design-audit`
  - `/demo/orders/[id]/buyer`
  - `/demo/orders/[id]/seller`
  - `/demo/orders/[id]/admin`
- Layout is responsive (mobile + desktop)

**Files**

- `src/app/layout.tsx`, `src/app/page.tsx`, `src/app/demo/**`

**Blocked by:** —  
**Blocks:** #2, #5

**Suggested labels:** `epic:setup`, `good first issue`

---

## Issue #2 — Install Trustless Work Blocks toolchain and dependencies

**Description**  
Run the Blocks CLI and install all required peer dependencies. Document any manual steps in PR description.

**Tasks**

- [ ] Run `npx trustless-work init` (shadcn, `.twblocks.json`)
- [ ] Run `npx trustless-work add wallet-kit`
- [ ] Run `npx trustless-work escrows`
- [ ] Install peer deps: `@trustless-work/escrow`, `@tanstack/react-query`, `@tanstack/react-query-devtools`, `@tanstack/react-table`, `@creit.tech/stellar-wallets-kit`, `react-hook-form`, `@hookform/resolvers`, `zod`, `axios`, `sonner`
- [ ] Update `.env.local.example` with `NEXT_PUBLIC_USE_MAINNET`, `NEXT_PUBLIC_USE_MOCK`

**Acceptance criteria**

- All packages in `package.json` dependencies
- `.twblocks.json` exists
- Escrow block components present under `src/components/escrows/` (or path chosen by CLI)
- shadcn `components.json` exists
- `pnpm install && pnpm build` succeeds

**Files**

- `package.json`, `pnpm-lock.yaml`, `components.json`, `.twblocks.json`, `.env.local.example`

**Blocked by:** #1  
**Blocks:** #3, #24

**Suggested labels:** `epic:setup`, `epic:blocks`, `priority:must-have`

---

## Issue #3 — App providers (QueryClient + TrustlessWorkConfig)

**Description**  
Create `src/components/Providers.tsx` with correct nesting. Wire into root layout without making the entire layout a Client Component.

**Tasks**

- [ ] Create `Providers.tsx` with `"use client"`
- [ ] `QueryClient` at module scope with sensible defaults (`staleTime`, `retry`)
- [ ] `TrustlessWorkConfig` with `development` / `mainNet` from env
- [ ] Include `ReactQueryDevtools` in development only
- [ ] Import `Providers` from server `layout.tsx`

**Acceptance criteria**

- Provider order: `QueryClientProvider` → `TrustlessWorkConfig` → `WalletProvider`
- No "No QueryClient set" runtime errors
- API key read from `NEXT_PUBLIC_API_KEY`

**Files**

- `src/components/Providers.tsx`, `src/app/layout.tsx`

**Blocked by:** #2, #4  
**Blocks:** #24, #32

**Suggested labels:** `epic:setup`, `priority:must-have`

---

## Issue #4 — WalletProvider (Stellar Wallets Kit)

**Description**  
Implement wallet context using `@creit.tech/stellar-wallets-kit` (Freighter, Albedo, xBull). Expose `kit`, `address`, `connect`, `disconnect`, `isConnected`.

**Tasks**

- [ ] Create `src/providers/WalletProvider.tsx`
- [ ] Kit at module scope; network matches `NEXT_PUBLIC_USE_MAINNET`
- [ ] Export `useWallet()` hook with context guard
- [ ] Integrate into `Providers.tsx` (innermost)

**Acceptance criteria**

- Wallet modal opens on connect
- Address available to child components after connect
- Testnet default when `NEXT_PUBLIC_USE_MAINNET=false`

**Files**

- `src/providers/WalletProvider.tsx`, `src/components/Providers.tsx`

**Blocked by:** #2  
**Blocks:** #3, #29, #31

**Suggested labels:** `epic:setup`, `priority:must-have`

---

## Issue #5 — Feature folder scaffold

**Description**  
Create the vertical slice directory structure per FOLDER_LAYOUT. Add barrel exports only where they improve DX.

**Tasks**

- [ ] Create `src/features/payment-widget/` subfolders: `components`, `hooks`, `services`, `schemas`, `utils`, `constants`
- [ ] Create `src/features/demo/` subfolders: `views`, `components`, `constants`
- [ ] Create `src/types/` with placeholder files
- [ ] Ensure `@/*` path alias resolves (already in `tsconfig.json`)

**Acceptance criteria**

- Folder structure matches [Target repository structure](#target-repository-structure)
- No domain logic in `src/app/` except route composition
- ESLint passes on empty placeholder exports if added

**Files**

- `src/features/**`, `src/types/**`

**Blocked by:** #1  
**Blocks:** #6, all feature issues

**Suggested labels:** `epic:setup`, `good first issue`

---

## Issue #6 — Shared TypeScript types

**Description**  
Define types from `PRODUCT-BRIEF.md` §18. Single source of truth for widget props, escrow status, roles, fees, callbacks, evidence.

**Tasks**

- [ ] `src/types/payment-widget.ts` — `PaymentWidgetProps`, `PaymentWidgetCallbacks`, modes
- [ ] `src/types/escrow.ts` — `PaymentEscrow`, `PaymentEscrowStatus`, `PaymentEvidence`
- [ ] `src/types/roles.ts` — `PaymentEscrowRoles`
- [ ] `src/types/fees.ts` — `PaymentEscrowFees`
- [ ] Export status union; no magic strings in components

**Acceptance criteria**

- Types exported from `src/types/`
- Strict TypeScript; no `any`
- Aligns with `PRODUCT-BRIEF.md` §18

**Files**

- `src/types/*.ts`

**Blocked by:** #5  
**Blocks:** Epic 2, Epic 3, Epic 4

**Suggested labels:** `epic:setup`, `good first issue`

---

# Epic 2 — Payment Widget UI Shell (custom components)

> **Goal:** Checkout-style UI that wraps Blocks — timeline, fees, roles, receipts.  
> **Labels:** `epic:widget-ui`, `priority:must-have`  
> **Blocked by:** Epic 1 (#5, #6)

---

## Issue #7 — `PaymentWidgetCard` layout component

**Description**  
Card shell for the payment widget: headline, title, amount, description, helper copy. Use shadcn `Card` composition.

**Acceptance criteria**

- Props: `title`, `description`, `amount`, `currency`, `children`
- Responsive; light/dark mode via semantic tokens
- Copy defaults per `PRODUCT-BRIEF.md` §23 ("Pay with Escrow", helper text)

**Files**

- `src/features/payment-widget/components/PaymentWidgetCard.tsx`

**Blocked by:** #2, #6  
**Blocks:** #12

---

## Issue #8 — `PaymentStatusTracker` timeline

**Description**  
Visual timeline: Created → Funded → Delivered → Approved → Released.

**Acceptance criteria**

- Shows completed, active, and pending steps
- Handles `error`, `disputed`, `cancelled` states
- Mobile-friendly (vertical stack)
- Labels from `constants/status-labels.ts`

**Files**

- `src/features/payment-widget/components/PaymentStatusTracker.tsx`
- `src/features/payment-widget/constants/status-labels.ts`
- `src/features/payment-widget/constants/timeline-steps.ts`

**Blocked by:** #6  
**Blocks:** #12, #18

---

## Issue #9 — `FeeBreakdown` component

**Description**  
Display payment amount, platform fee, Trustless Work fee, seller net amount.

**Acceptance criteria**

- Props: `amount`, `platformFeeBps`, `trustlessWorkFeeBps`, `currency`
- Uses `calculateFees` util (Issue #13)
- Handles zero platform fee
- Example output per `PRODUCT-BRIEF.md` §16

**Files**

- `src/features/payment-widget/components/FeeBreakdown.tsx`

**Blocked by:** #13  
**Blocks:** #12

---

## Issue #10 — `RoleSummary` component

**Description**  
Human-readable role mapping with shortened Stellar addresses. Used in admin/debug mode.

**Acceptance criteria**

- Shows payer, receiver, milestone marker, approver, release signer, platform
- Uses `shortenAddress` util
- Plain-language labels (not blockchain jargon)
- Optional `debug` prop shows raw addresses

**Files**

- `src/features/payment-widget/components/RoleSummary.tsx`

**Blocked by:** #6, #14  
**Blocks:** #23, #12

---

## Issue #11 — `ErrorState` and `SuccessReceipt` components

**Description**  
Terminal UI states for failed actions and successful release.

**Acceptance criteria**

- `ErrorState`: message, optional retry action
- `SuccessReceipt`: amount, escrow ID, optional tx hash placeholder
- shadcn `Alert` / `Card`; accessible markup

**Files**

- `src/features/payment-widget/components/ErrorState.tsx`
- `src/features/payment-widget/components/SuccessReceipt.tsx`

**Blocked by:** #2, #6  
**Blocks:** #12, #29

---

## Issue #12 — `TrustlessPaymentWidget` shell (no blocks yet)

**Description**  
Main orchestrator component accepting minimal props. Renders card, status, fees. Supports `readOnly` and mock flag prop — **blocks wired in Epic 6**.

**Acceptance criteria**

- Accepts `PaymentWidgetProps` from types
- Renders `PaymentWidgetCard`, `PaymentStatusTracker`, `FeeBreakdown`
- `readOnly` hides action area
- `mode`: `create` | `existing` | `status` affects layout slots (placeholders OK)
- Named export; arrow function component

**Files**

- `src/features/payment-widget/components/TrustlessPaymentWidget.tsx`

**Blocked by:** #7, #8, #9  
**Blocks:** #18, #25–#30, #32

---

# Epic 3 — Utils & Services

> **Goal:** Pure helpers and API→widget mapping. No JSX.  
> **Labels:** `epic:services`, `priority:must-have`  
> **Blocked by:** #6

---

## Issue #13 — Fee and formatting utilities

**Description**

- `calculateFees(amount, platformFeeBps, trustlessWorkFeeBps)`
- `formatCurrency(amount, currency)`
- `shortenAddress(address, chars?)`

**Acceptance criteria**

- Unit-testable pure functions
- BPS math correct (250 bps = 2.5%)
- Named exports with `function` keyword for non-UI utils

**Files**

- `src/features/payment-widget/utils/calculateFees.ts`
- `src/features/payment-widget/utils/formatCurrency.ts`
- `src/features/payment-widget/utils/shortenAddress.ts`

**Blocked by:** #6  
**Blocks:** #9, #16

---

## Issue #14 — `mapEscrowStatus` service

**Description**  
Map Trustless Work indexer/API escrow object → `PaymentEscrowStatus` for the widget timeline and CTAs.

**Acceptance criteria**

- Handles single-release, single-milestone MVP
- Maps: unfunded, funded, delivered (status text), approved, released, disputed
- Returns user-facing label key or status enum
- Document mapping table in code comment or PR

**Files**

- `src/features/payment-widget/services/mapEscrowStatus.ts`

**Blocked by:** #6  
**Blocks:** #17, #30, #33

---

## Issue #15 — `buildEscrowPayload` service

**Description**  
Map `PaymentWidgetProps` → `InitializeSingleReleaseEscrowPayload` for SDK/blocks.

**Acceptance criteria**

- Converts `platformFeeBps` → `platformFee` number for API
- Default single milestone for MVP
- Validates required addresses and trustline
- `engagementId` from `externalReference` or generated id
- Types from `@trustless-work/escrow/types`

**Files**

- `src/features/payment-widget/services/buildEscrowPayload.ts`

**Blocked by:** #6  
**Blocks:** #25

---

# Epic 4 — Mock Mode

> **Goal:** Full escrow flow without API keys or wallet.  
> **Labels:** `epic:mock`, `priority:must-have`  
> **Blocked by:** Epic 2 #12, Epic 3

---

## Issue #16 — Mock escrow state machine hook

**Description**  
`useMockPaymentEscrow` — local state transitions: create → deposit → deliver → approve → release.

**Acceptance criteria**

- Full flow works without `NEXT_PUBLIC_API_KEY`
- Generates mock `escrowId` and tx hashes
- Exposes actions matching real mode interface for orchestrator
- State aligns with `PaymentEscrowStatus`

**Files**

- `src/features/payment-widget/hooks/useMockPaymentEscrow.ts`
- `src/lib/mock/mockEscrows.ts` (optional shared store)

**Blocked by:** #6, #14  
**Blocks:** #18

---

## Issue #17 — Mock demo data

**Description**  
Static mock order: **Premium Design Audit — 500 USDC** with buyer/seller/platform addresses and role presets.

**Acceptance criteria**

- `mock-order.ts` with order id, product, amount, roles
- Reusable across demo views
- Matches `PRODUCT-BRIEF.md` §26 recommended demo

**Files**

- `src/features/demo/constants/mock-order.ts`
- `src/lib/mock/mockOrders.ts`, `src/lib/mock/mockUsers.ts` (if split)

**Blocked by:** #5  
**Blocks:** Epic 5

**Suggested labels:** `epic:mock`, `good first issue`

---

## Issue #18 — Wire mock mode into `TrustlessPaymentWidget`

**Description**  
When `useMock` prop or `NEXT_PUBLIC_USE_MOCK=true`, use mock hook instead of blocks/SDK.

**Acceptance criteria**

- Toggle between mock and real without code changes (env or prop)
- Timeline and fees update on each mock action
- Callbacks fire (`onEscrowCreated`, `onDepositConfirmed`, etc.)
- Blocks components not rendered in mock mode

**Files**

- `src/features/payment-widget/components/TrustlessPaymentWidget.tsx`
- `src/features/payment-widget/hooks/usePaymentWidget.ts` (stub OK, full in #32)

**Blocked by:** #12, #16  
**Blocks:** Epic 5 demo pages

---

# Epic 5 — Demo App

> **Goal:** Realistic checkout context for the widget.  
> **Labels:** `epic:demo`, `priority:must-have`  
> **Blocked by:** #18, #17

---

## Issue #19 — Demo layout and landing page

**Description**  
Home page explains the template and links to demo flows. Shared demo layout optional (nav between personas).

**Acceptance criteria**

- Landing describes pay → hold → deliver → approve → release
- Links to product page and sample order routes
- Responsive

**Files**

- `src/app/page.tsx`
- `src/features/demo/components/DemoNav.tsx` (optional)

**Blocked by:** #1  
**Blocks:** #20–#23

---

## Issue #20 — Product demo page

**Route:** `/demo/product/design-audit`

**Description**  
Product/service details + embedded `TrustlessPaymentWidget` in `create` mode.

**Acceptance criteria**

- Product title, seller info, price 500 USDC
- Widget in `mode="create"`
- View composes from `ProductDemoView.tsx`; route file is thin

**Files**

- `src/app/demo/product/[slug]/page.tsx`
- `src/features/demo/views/ProductDemoView.tsx`

**Blocked by:** #18, #17

---

## Issue #21 — Buyer order demo page

**Route:** `/demo/orders/[id]/buyer`

**Acceptance criteria**

- Order summary
- Widget in `existing` or `status` mode with buyer actions (deposit, approve, release)
- Shows evidence when seller submitted (mock data)

**Files**

- `src/app/demo/orders/[id]/buyer/page.tsx`
- `src/features/demo/views/BuyerOrderView.tsx`

**Blocked by:** #18

---

## Issue #22 — Seller order demo page

**Route:** `/demo/orders/[id]/seller`

**Acceptance criteria**

- Funding status visible
- Seller can mark delivered (mock or block)
- Evidence URL field (should-have: full form in #40)

**Files**

- `src/app/demo/orders/[id]/seller/page.tsx`
- `src/features/demo/views/SellerOrderView.tsx`

**Blocked by:** #18

---

## Issue #23 — Admin order demo page

**Route:** `/demo/orders/[id]/admin`

**Acceptance criteria**

- Full role mapping via `RoleSummary`
- Fee breakdown
- Escrow JSON debug preview
- Platform release mode toggle documented in UI copy

**Files**

- `src/app/demo/orders/[id]/admin/page.tsx`
- `src/features/demo/views/AdminOrderView.tsx`

**Blocked by:** #10, #18

---

# Epic 6 — Escrow Blocks Integration

> **Goal:** Real on-chain flow via Blocks + SDK.  
> **Labels:** `epic:blocks`, `priority:must-have`  
> **Blocked by:** Epic 1 #2–#4, Issue #12

---

## Issue #24 — Verify installed blocks and document paths

**Description**  
Audit CLI output: list exported components, confirm import paths, add short README in `src/components/escrows/` if missing.

**Acceptance criteria**

- Document which block files exist and their exports
- Confirm `useEscrowContext` available
- No broken imports after `pnpm build`

**Files**

- `src/components/escrows/` (+ optional README)

**Blocked by:** #2  
**Blocks:** #25–#31

---

## Issue #25 — Integrate `InitializeEscrowForm` in create mode

**Description**  
Wire block into widget `mode="create"`. Map props via `buildEscrowPayload`.

**Acceptance criteria**

- Create flow uses block form (not custom duplicate form)
- Single-release, one milestone
- `onEscrowCreated` callback fires with normalized escrow
- Wallet connect required before create

**Files**

- `src/features/payment-widget/components/TrustlessPaymentWidget.tsx`

**Blocked by:** #12, #15, #24, #3  
**Blocks:** #30

---

## Issue #26 — Integrate `ChangeMilestoneStatus` (seller deliver)

**Description**  
Show block when connected wallet is service provider and escrow is funded.

**Acceptance criteria**

- Only visible to milestone marker role
- Calls `setSelectedEscrow` before render
- `onDelivered` callback fires

**Blocked by:** #24, #30  
**Blocks:** #32

---

## Issue #27 — Integrate `ApproveMilestone`

**Description**  
Approver dialog with confirmation (approval is irreversible).

**Acceptance criteria**

- Confirm dialog before approve
- Only visible to approver role after delivery
- `onApproved` callback fires

**Blocked by:** #24, #30  
**Blocks:** #32

---

## Issue #28 — Integrate `ReleaseEscrow`

**Description**  
Release signer action after approval.

**Acceptance criteria**

- Only available when milestone approved and not released
- Shows `SuccessReceipt` on completion
- `onReleased` callback with tx reference

**Blocked by:** #24, #30, #11  
**Blocks:** #32

---

## Issue #29 — `FundEscrowAction` custom component

**Description**  
No dedicated Fund block in Blocks catalog — build thin UI using `useFundEscrow` + `useSendTransaction` + wallet sign.

**Acceptance criteria**

- Funder role only; awaiting deposit state
- XDR sign → submit pattern
- `onDepositStarted`, `onDepositConfirmed` callbacks
- Error states via `ErrorState`

**Files**

- `src/features/payment-widget/components/FundEscrowAction.tsx`

**Blocked by:** #3, #4, #11, #24  
**Blocks:** #32

**Suggested labels:** `epic:blocks`, `help wanted`

---

## Issue #30 — Escrow context wiring for `existing` and `status` modes

**Description**  
On mount with `escrowId`, fetch escrow (indexer hook) and call `setSelectedEscrow`. Support read-only `status` mode.

**Acceptance criteria**

- `mode="existing"`: fetch + select + show role-appropriate blocks
- `mode="status"` + `readOnly`: timeline + fees only
- Invalid/missing escrow shows `ErrorState`

**Blocked by:** #14, #24, #33  
**Blocks:** #26–#28

---

## Issue #31 — Integrate `WalletButton`

**Description**  
Add Blocks wallet button to widget header or demo views.

**Acceptance criteria**

- Connect/disconnect works with `WalletProvider`
- Widget actions disabled until connected (real mode)

**Blocked by:** #4, #24  
**Blocks:** #35

---

# Epic 7 — Orchestration & Real-Mode Hooks

> **Goal:** Single hook coordinates mock vs real, roles, callbacks.  
> **Labels:** `epic:widget-ui`, `priority:must-have`  
> **Blocked by:** Epic 4, Epic 6

---

## Issue #32 — `usePaymentWidget` hook

**Description**  
Central orchestration: mode, mock vs real, derived status, available actions, callback wiring.

**Acceptance criteria**

- Branches on `NEXT_PUBLIC_USE_MOCK` / prop
- Returns `{ status, fees, actions, escrow, isLoading, error }`
- No JSX in hook

**Files**

- `src/features/payment-widget/hooks/usePaymentWidget.ts`

**Blocked by:** #18, #25–#29  
**Blocks:** #34, #35

---

## Issue #33 — `usePaymentEscrow` data hook (TanStack Query)

**Description**  
Fetch escrow by `contractId` via `useGetEscrowFromIndexerByContractIds` (or equivalent SDK read hook).

**Acceptance criteria**

- Query key includes contractId and network
- Maps result through `mapEscrowStatus`
- Supports manual refetch after mutations
- Handles loading and error states

**Files**

- `src/features/payment-widget/hooks/usePaymentEscrow.ts`

**Blocked by:** #14, #3  
**Blocks:** #30

---

## Issue #34 — Widget callbacks implementation

**Description**  
Fire all `PaymentWidgetCallbacks` at the correct lifecycle points (mock + real).

**Acceptance criteria**

- All callbacks in `PRODUCT-BRIEF.md` §18 implemented
- `onError` receives typed `PaymentWidgetError`
- Callbacks optional (no crash if omitted)

**Blocked by:** #32  
**Blocks:** MVP sign-off

---

## Issue #35 — Role-based action visibility

**Description**  
Compare connected wallet address to escrow roles; show/hide block actions accordingly.

**Acceptance criteria**

- Buyer sees deposit/approve/release (if release signer)
- Seller sees deliver when funded
- Platform sees release when configured as release signer
- Wrong role sees read-only status with clear copy

**Blocked by:** #31, #32  
**Blocks:** MVP sign-off

---

# Epic 8 — Documentation

> **Goal:** Enable fork-and-ship adoption.  
> **Labels:** `epic:docs`, `priority:must-have`

---

## Issue #36 — Component API documentation

**File:** `docs/component-api.md`

**Include**

- Minimal and full usage examples
- All props, callbacks, modes
- Mock vs real configuration
- Theming notes (should-have)

**Blocked by:** #12, #32

---

## Issue #37 — Role mapping documentation

**File:** `docs/role-mapping.md`

**Include**

- Default buyer-release model
- Platform-release alternative
- Trustless Work role → widget role table
- Security: irreversible approve, trustlines, G vs C address

**Blocked by:** — (can start early)

---

## Issue #38 — Demo script documentation

**File:** `docs/demo-script.md`

**Include**

- 3-minute walkthrough script
- Persona switching (buyer → seller → admin)
- Mock mode steps vs testnet steps

**Blocked by:** Epic 5

---

## Issue #39 — Update root README

**Description**  
Align README with Blocks setup CLI, env vars, folder structure, link to `BUILD-PLAN.md`.

**Acceptance criteria**

- Quickstart uses `npx trustless-work init`
- Lists peer dependencies
- Links to demo routes and docs

**Files**

- `README.md`

**Blocked by:** #2, Epic 5

---

# Epic 9 — Should-Have Polish (v1)

> **Goal:** Items from `PRODUCT-BRIEF.md` §8 should-have.  
> **Labels:** `priority:should-have`

---

## Issue #40 — Delivery evidence form

**Description**  
Zod schema + react-hook-form for evidence URL (tracking, GitHub PR, Figma, etc.) integrated with deliver flow.

**Files**

- `src/features/payment-widget/schemas/evidence.schema.ts`
- `src/features/payment-widget/components/EvidenceForm.tsx`

**Blocked by:** #26

---

## Issue #41 — Widget theming props

**Description**  
Support `theme={{ variant, radius }}` prop from brief — map to shadcn/CSS variables where possible.

**Blocked by:** #12

---

## Issue #42 — Copyable escrow ID and transaction hash display

**Description**  
Show escrow ID and tx hashes after deposit/release with copy-to-clipboard.

**Blocked by:** #11, #28, #29

---

## Issue #43 — Webhook / event integration notes

**File:** `docs/extension-ideas.md` (section only)

**Description**  
Document how platforms listen to escrow events off-chain (no implementation required).

**Blocked by:** —

---

# MVP completion checklist

From `PRODUCT-BRIEF.md` §24 — track at epic level:

| # | Criterion | Primary issues |
| --- | --- | --- |
| 1 | Clone and run demo locally | #1, #2, #19 |
| 2 | Mock mode without API keys | #16, #18 |
| 3 | Clean React component API | #6, #12, #36 |
| 4 | Buyer create + fund (mock) | #16, #21 |
| 5 | Seller mark delivered (mock) | #16, #22 |
| 6 | Buyer approve (mock) | #16, #21 |
| 7 | Release signer release (mock) | #16, #21 |
| 8 | Fee breakdown visible | #9, #13 |
| 9 | Role mapping in admin/debug | #10, #23 |
| 10 | Documentation for adaptation | Epic 8 |
| 11 | TW integration isolated | Epic 6, Epic 3 services |
| 12 | Starter for checkout/marketplace/agency | #20–#23, #36 |

**Real-mode MVP (testnet):** Epic 6 + #32–#35 complete the on-chain path.

---

# Out of scope (do not open issues without approval)

Per `PRODUCT-BRIEF.md` §8 and §25:

- Multi-item cart, multi-seller checkout
- Full dispute resolution UI
- Full refund flow
- Fiat / card payments
- Inventory, user accounts, messaging, notifications
- Multi-milestone escrows (MVP = one milestone)
- npm package publish (v2)

---

# Creating a GitHub issue (template)

**Before creating:** confirm this issue's phase in [Issue creation order](#issue-creation-order-for-agents--maintainers). All `Blocked by` issues must already exist on GitHub.

Copy into a new issue:

```markdown
## Summary
[Brief description from issue above]

## Epic
Epic N — [name]

## BUILD-PLAN ID
Issue #N (logical ID in docs/BUILD-PLAN.md)

## Creation phase
P? — see docs/BUILD-PLAN.md#issue-creation-order-for-agents--maintainers

## Dependencies
- Blocked by: #[GitHub-N], #[GitHub-M]  ← real GitHub issue numbers only
- Blocks: #[GitHub-X]

## Tasks
- [ ] ...
- [ ] ...

## Acceptance criteria
- [ ] ...

## Files (expected)
- `path/to/files`

## References
- docs/BUILD-PLAN.md — Issue #N
- docs/PRODUCT-BRIEF.md — §X
```

---

# Suggested milestone grouping

Same milestones as [Mapping phases to milestones](#mapping-phases-to-milestones) — use when labeling GitHub Milestones.

| Milestone | Create phases | BUILD-PLAN issues | Outcome |
| --- | --- | --- | --- |
| **M1 — Foundation** | P1–P4 | `#1`–`#6` | Toolchain, providers, types, folders |
| **M2 — Widget shell + mock** | P5–P8 | `#7`–`#18` | Demo-able mock flow end-to-end |
| **M3 — Demo app** | P9 | `#19`–`#23` | All four demo pages |
| **M4 — Blocks real mode** | P10–P12 | `#24`–`#31` | Testnet escrow operations |
| **M5 — Orchestration** | P13–P14 | `#32`–`#35` | Production-ready widget API |
| **M6 — Docs & polish** | P15–P16 | `#36`–`#43` | OSS-ready documentation |

---

*Last updated: aligned with `PRODUCT-BRIEF.md`, FOLDER_LAYOUT vertical slices, `@trustless-work/blocks` compose strategy, and phased issue creation order (P1–P16).*
