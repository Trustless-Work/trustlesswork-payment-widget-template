# Build Plan & OSS Issue Backlog

This document is the **implementation source of truth** for contributors building the Trustless Work Payment Widget template. Use it to understand architecture, scope, and to **create GitHub issues** one-to-one from the sections below.

Each issue is sized for **meaningful contributor effort** (typically 1–4 days), not micro-tasks. Prefer one merged PR per issue where possible.

**Related docs**

| Document | Purpose |
| --- | --- |
| [`PRODUCT-BRIEF.md`](./PRODUCT-BRIEF.md) | Product concept, widget API, UX, MVP scope, acceptance criteria |
| [`.cursor/rules/FOLDER_LAYOUT.mdc`](../.cursor/rules/FOLDER_LAYOUT.mdc) | Vertical slice architecture (`src/features/`) |
| [Trustless Work Blocks docs](https://docs.trustlesswork.com/trustless-work/escrow-blocks-sdk/getting-started) | Escrow Blocks SDK |
| [`.agents/skills/trustless-work/`](../.agents/skills/trustless-work/) | Local skill: SDK, Blocks, XDR signing, roles |

---

## How to use this document

1. **Creating GitHub issues?** Follow [Issue creation order](#issue-creation-order-for-agents--maintainers) — create **#1 → #11** in phase order.
2. **Implementing work?** Only pick issues whose **Blocked by** dependencies are closed/merged.
3. **Copy the issue template** at the bottom into GitHub Issues.
4. **Follow architecture rules** — routing stays thin; domain code lives under `src/features/`.
5. **Cross-check MVP scope** in `PRODUCT-BRIEF.md` §8 and §24 before expanding scope.

### Effort scale

| Size | Typical effort | Meaning |
| --- | --- | --- |
| **M** | 1–2 days | Focused feature slice; one domain area |
| **L** | 2–3 days | Multi-component or multi-file; integration work |
| **XL** | 3–5 days | Cross-cutting; blocks + SDK + hooks; testnet verification |

### Suggested GitHub labels

| Label | Use for |
| --- | --- |
| `epic:setup` | Issues #1–#2 |
| `epic:widget-ui` | Issues #3–#5, #9 |
| `epic:demo` | Issue #6 |
| `epic:blocks` | Issues #7–#8 |
| `epic:docs` | Issue #10 |
| `priority:must-have` | Issues #1–#10 (MVP) |
| `priority:should-have` | Issue #11 |
| `size:M` / `size:L` / `size:XL` | Effort estimate |
| `help wanted` | Ready for external contributors |

---

## Architecture decisions

### 1. Vertical slices (`src/features/`)

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

| MVP step | Escrow Block / SDK | Custom in this repo |
| --- | --- | --- |
| Create escrow | `InitializeEscrowForm` | Payload mapping from widget props |
| Deposit funds | `useFundEscrow` (SDK hook) | **Custom UI** — no dedicated block listed |
| Mark delivered | `ChangeMilestoneStatus` | Evidence form (Issue #11) |
| Approve | `ApproveMilestone` | Role gating in orchestrator |
| Release | `ReleaseEscrow` | Role gating in orchestrator |
| List / select escrow | `EscrowsByRole`, `EscrowsBySigner` | `setSelectedEscrow` wiring |
| Connect wallet | `WalletButton` | `WalletProvider` (Stellar Wallets Kit) |

### 3. Provider order (non-negotiable)

```
QueryClientProvider → TrustlessWorkConfig → WalletProvider → {children}
```

### 4. Mock mode vs real mode

| Mode | Trigger | Behavior |
| --- | --- | --- |
| **Mock** | `NEXT_PUBLIC_USE_MOCK=true` or widget prop | Local state machine; blocks hidden; no API key |
| **Real** | Mock off + API key set | Blocks + SDK hooks + wallet signing |

---

## Tech stack & dependencies

```bash
pnpm add @trustless-work/blocks @trustless-work/escrow \
  @tanstack/react-query @tanstack/react-query-devtools @tanstack/react-table \
  @creit.tech/stellar-wallets-kit \
  react-hook-form @hookform/resolvers zod axios sonner
```

```bash
npx trustless-work init
npx trustless-work add wallet-kit
npx trustless-work escrows
```

See [Target repository structure](#target-repository-structure) for full folder layout.

---

## Target repository structure

```
src/
├── app/                          # Thin routes → demo views
├── components/ui/                # shadcn
├── components/Providers.tsx
├── components/escrows/           # CLI-installed blocks
├── providers/WalletProvider.tsx
├── types/                        # payment-widget, escrow, roles, fees
└── features/
    ├── payment-widget/           # Widget slice
    └── demo/                     # Demo slice
```

---

## Payment widget state machine (reference)

```
idle → creating_escrow → awaiting_deposit → funded → awaiting_delivery
  → delivered → awaiting_approval → approved → ready_to_release
  → releasing → released
```

Additional UI states: `cancelled`, `disputed`, `expired`, `error`.

---

## Issue dependency graph

```
#1 Foundation & scaffold
    │
    ▼
#2 Blocks toolchain & providers
    │
    ├──────────────────┐
    ▼                  ▼
#3 Domain layer    #4 Widget UI shell
    │                  │
    └────────┬─────────┘
             ▼
        #5 Mock mode (full flow)
             │
             ▼
        #6 Demo application
             │
             ▼
        #7 Blocks: create, fund, wallet, fetch
             │
             ▼
        #8 Blocks: deliver, approve, release
             │
             ▼
        #9 Orchestration & public API
             │
             ▼
        #10 Documentation package
             │
             ▼
        #11 v1 polish (should-have)
```

---

## Issue creation order (for agents & maintainers)

> Issue numbers `#1`–`#11` are **logical BUILD-PLAN IDs**. On GitHub, link **real issue numbers** in `Blocked by`.

### Rules

1. **Create in order P1 → P11** — each phase depends on the previous one.
2. **One GitHub issue per BUILD-PLAN entry** — do not split further without maintainer approval.
3. **Do not create #7+ before #5 and #6 are on GitHub** if the goal is a mock-first MVP.
4. **Issue #11 last** — should-have polish only.

### Phase sequence

| Phase | Create | Effort | Outcome |
| --- | --- | --- | --- |
| **P1** | `#1` | M | Runnable app + folders + types + routes |
| **P2** | `#2` | L | Blocks CLI, shadcn, wallet, providers |
| **P3** | `#3` | M | Utils, services, status mapping |
| **P4** | `#4` | L | Full widget UI shell |
| **P5** | `#5` | L | End-to-end mock flow in widget |
| **P6** | `#6` | L | Complete demo app (4 personas) |
| **P7** | `#7` | XL | Real mode: create, deposit, wallet, fetch |
| **P8** | `#8` | L | Real mode: lifecycle blocks |
| **P9** | `#9` | L | Orchestrator hook + callbacks + roles |
| **P10** | `#10` | M | All MVP documentation |
| **P11** | `#11` | M | Should-have polish |

### Compact checklist (for agents)

```
P1:  #1
P2:  #2
P3:  #3
P4:  #4
P5:  #5
P6:  #6
P7:  #7
P8:  #8
P9:  #9
P10: #10
P11: #11
```

### Milestones

| Milestone | Issues | Demo milestone |
| --- | --- | --- |
| **M1 — Foundation** | #1, #2 | App runs with providers |
| **M2 — Widget + mock** | #3, #4, #5 | Full mock flow in widget |
| **M3 — Demo app** | #6 | Walkthrough all personas |
| **M4 — Testnet** | #7, #8, #9 | Real escrow on testnet |
| **M5 — Ship** | #10, #11 | Docs + polish |

### Agent prompt snippet

> Create **11 GitHub issues** from `docs/BUILD-PLAN.md` in order **#1 → #11**. Each issue is intentionally large (1–4 days). Set `Blocked by` to prior GitHub issue numbers only. For mock-first MVP, complete through **#6** before assigning **#7**.

---

# Issue backlog

---

## Issue #1 — App foundation, feature scaffold & shared types

| | |
| --- | --- |
| **Epic** | Project setup |
| **Effort** | M (1–2 days) |
| **Labels** | `epic:setup`, `priority:must-have`, `size:M` |
| **Blocked by** | — |
| **Blocks** | #2, #3, #4, #6 |

### Summary

Establish the application skeleton: working Next.js app, demo route placeholders, vertical slice folder layout, and the full TypeScript contract layer for the payment widget. This issue delivers the **structural baseline** every other issue builds on.

### Scope

**In scope**

- Verify and clean up Next.js 16 + TypeScript strict + Tailwind v4 scaffold
- Demo route placeholders (thin pages, "coming soon" OK initially)
- Minimal landing page with links to demo flows
- Complete `src/features/` and `src/types/` scaffold per FOLDER_LAYOUT
- All shared types from `PRODUCT-BRIEF.md` §18

**Out of scope**

- Trustless Work packages (Issue #2)
- Widget components (Issue #4)
- Real demo content (Issue #6)

### Tasks

- [ ] Confirm `pnpm dev`, `pnpm build`, `pnpm lint` pass
- [ ] Update root `layout.tsx` metadata (title, description, lang)
- [ ] Replace create-next-app homepage with landing: product pitch + links to demo routes
- [ ] Add thin route files:
  - `/demo/product/[slug]`
  - `/demo/orders/[id]/buyer`, `/seller`, `/admin`
- [ ] Create folder tree under `src/features/payment-widget/` and `src/features/demo/`
- [ ] Create `src/types/payment-widget.ts` — props, callbacks, modes
- [ ] Create `src/types/escrow.ts` — `PaymentEscrow`, `PaymentEscrowStatus`, evidence
- [ ] Create `src/types/roles.ts`, `src/types/fees.ts`
- [ ] Add `src/features/payment-widget/constants/status-labels.ts` and `timeline-steps.ts` (enums/labels only)

### Acceptance criteria

- [ ] App runs at `http://localhost:3000` with responsive landing
- [ ] All demo routes resolve without 404
- [ ] Folder structure matches [Target repository structure](#target-repository-structure)
- [ ] Types exported from `src/types/`; strict TS; no `any`
- [ ] No domain logic in `src/app/` beyond route composition
- [ ] PR includes brief note on `@/*` path alias usage

### Definition of done

- New contributor can clone, install, run dev, and navigate all routes
- Types are importable from `@/types/*` and ready for Issue #4

### Files (expected)

- `src/app/layout.tsx`, `src/app/page.tsx`, `src/app/demo/**`
- `src/features/**` (empty scaffold)
- `src/types/**`
- `src/features/payment-widget/constants/**`

---

## Issue #2 — Trustless Work Blocks toolchain, shadcn & app providers

| | |
| --- | --- |
| **Epic** | Project setup |
| **Effort** | L (2–3 days) |
| **Labels** | `epic:setup`, `epic:blocks`, `priority:must-have`, `size:L` |
| **Blocked by** | #1 |
| **Blocks** | #4, #5, #7, #8, #9 |

### Summary

Bootstrap the full Trustless Work integration stack: Blocks CLI, shadcn/ui, peer dependencies, Stellar Wallets Kit, and the three-layer provider tree. After this issue, the app is ready to render Blocks and sign transactions on testnet.

### Scope

**In scope**

- `npx trustless-work init`, `add wallet-kit`, `escrows`
- All peer dependencies in `package.json`
- `WalletProvider` with Freighter / Albedo / xBull
- `Providers.tsx` with correct nesting + React Query Devtools (dev only)
- Updated `.env.local.example`
- Short README section in PR for env vars

**Out of scope**

- Composing blocks into payment widget (Issues #7–#8)
- Custom widget UI (Issue #4)

### Tasks

- [ ] Run Blocks CLI; commit `components.json`, `.twblocks.json`, escrow block files
- [ ] Install: `@trustless-work/blocks`, `@trustless-work/escrow`, TanStack packages, wallet kit, RHF, zod, axios, sonner
- [ ] Add shadcn primitives needed globally (e.g. `button`, `card`, `alert`, `dialog`, `sonner` toaster in layout)
- [ ] Implement `src/providers/WalletProvider.tsx` — `kit`, `address`, `connect`, `disconnect`, `isConnected`, `useWallet()`
- [ ] Implement `src/components/Providers.tsx` — module-scope `QueryClient`, `TrustlessWorkConfig`, nest wallet innermost
- [ ] Wire `Providers` from server `layout.tsx` (layout stays Server Component)
- [ ] Document escrow block paths in `src/components/escrows/README.md` (exports, import examples)
- [ ] Update `.env.local.example`: `NEXT_PUBLIC_API_KEY`, `NEXT_PUBLIC_USE_MAINNET`, `NEXT_PUBLIC_USE_MOCK`, optional issuer/platform addresses

### Acceptance criteria

- [ ] `pnpm install && pnpm build` succeeds
- [ ] Provider order verified: QueryClient → TrustlessWorkConfig → WalletProvider
- [ ] Wallet modal connects on testnet; address exposed via `useWallet()`
- [ ] No runtime "No QueryClient set" errors
- [ ] Escrow block files present and importable
- [ ] Sonner toasts render (smoke test)

### Definition of done

- A dev with API key and Freighter can load the app, connect wallet, and import any block component without provider errors

### Files (expected)

- `package.json`, `pnpm-lock.yaml`, `components.json`, `.twblocks.json`
- `src/components/Providers.tsx`, `src/components/ui/**`
- `src/providers/WalletProvider.tsx`, `src/components/escrows/**`
- `.env.local.example`, `src/app/layout.tsx`

---

## Issue #3 — Payment widget domain layer (utils & services)

| | |
| --- | --- |
| **Epic** | Services |
| **Effort** | M (1–2 days) |
| **Labels** | `epic:services`, `priority:must-have`, `size:M` |
| **Blocked by** | #1 |
| **Blocks** | #4, #5, #7, #8, #9 |

### Summary

Implement the **pure domain layer** between Trustless Work API shapes and the payment widget: fee math, formatting, status mapping, and escrow payload building. No React components in this issue — only testable logic.

### Scope

**In scope**

- Fee calculation (BPS → amounts)
- Currency and address formatting
- `mapEscrowStatus` — indexer/API escrow → widget state machine
- `buildEscrowPayload` — widget props → single-release initialize payload
- Input validation guards where needed

**Out of scope**

- UI components
- Mock state machine (Issue #5)
- TanStack Query hooks (Issue #7)

### Tasks

- [ ] `calculateFees(amount, platformFeeBps, trustlessWorkFeeBps)` — platform fee, TW fee, receiver net; zero-fee edge case
- [ ] `formatCurrency(amount, currency)` — USDC MVP
- [ ] `shortenAddress(address, chars?)`
- [ ] `mapEscrowStatus(escrow)` — document mapping table in file header; handle unfunded, funded, delivered text, approved, released, disputed
- [ ] `buildEscrowPayload(props)` — bps → `platformFee` number; single milestone; trustline object; role mapping; `engagementId`
- [ ] Optional: `assertPaymentWidgetInput(props)` for create mode required fields

### Acceptance criteria

- [ ] All functions are pure, strictly typed, exported with `function` keyword
- [ ] BPS math matches `PRODUCT-BRIEF.md` §16 example (500 USDC, 2.5% platform, 0.3% TW)
- [ ] `buildEscrowPayload` output satisfies `InitializeSingleReleaseEscrowPayload` types
- [ ] `mapEscrowStatus` covers all MVP states in `PRODUCT-BRIEF.md` §11
- [ ] No `any`; no React imports in services/utils

### Definition of done

- Issue #4 and #5 can import utils/services without duplicating business logic
- Mapping table is clear enough for a new contributor to extend

### Files (expected)

- `src/features/payment-widget/utils/**`
- `src/features/payment-widget/services/mapEscrowStatus.ts`
- `src/features/payment-widget/services/buildEscrowPayload.ts`

---

## Issue #4 — Payment widget UI shell (checkout experience)

| | |
| --- | --- |
| **Epic** | Widget UI |
| **Effort** | L (2–3 days) |
| **Labels** | `epic:widget-ui`, `priority:must-have`, `size:L` |
| **Blocked by** | #1, #2, #3 |
| **Blocks** | #5, #7, #9, #11 |

### Summary

Build the **complete custom UI shell** for the payment widget: checkout card, timeline, fee breakdown, role summary, error/receipt states, and the main `TrustlessPaymentWidget` orchestrator skeleton. Action slots are placeholders — mock and blocks wiring come in later issues.

### Scope

**In scope**

- All presentation components listed below
- `TrustlessPaymentWidget` accepting full `PaymentWidgetProps`
- Layout for three modes: `create`, `existing`, `status`
- `readOnly` and `debug` props
- Responsive + light/dark via semantic tokens
- Consumer-friendly copy per `PRODUCT-BRIEF.md` §23

**Out of scope**

- Mock state transitions (Issue #5)
- Escrow Blocks composition (Issues #7–#8)
- Full orchestrator hook (Issue #9)

### Tasks

- [ ] `PaymentWidgetCard` — headline, title, amount, description, helper copy, action slot
- [ ] `PaymentStatusTracker` — 5-step timeline; active/complete/pending; error/disputed/cancelled variants
- [ ] `FeeBreakdown` — wired to `calculateFees`; collapsible on mobile optional
- [ ] `RoleSummary` — human labels + shortened addresses; expanded debug view
- [ ] `ErrorState` — message, retry callback, shadcn Alert
- [ ] `SuccessReceipt` — amount, escrow ID, tx hash slot, release confirmation copy
- [ ] `TrustlessPaymentWidget` — composes above; mode-based layout; action area placeholder component
- [ ] Export public component from feature entry (e.g. `@/features/payment-widget/components/TrustlessPaymentWidget`)

### Acceptance criteria

- [ ] Widget renders with static/mock props in Storybook-like isolation (demo page or temp route OK)
- [ ] All subcomponents use shadcn primitives; no raw styled divs for alerts/empty states
- [ ] Mobile: timeline stacks vertically; card readable at 320px width
- [ ] `readOnly` hides action slot; `debug` shows `RoleSummary` expanded
- [ ] Copy uses "Funds secured", not blockchain jargon
- [ ] Named arrow function exports; strict props typing

### Definition of done

- Maintainer can render `<TrustlessPaymentWidget {...sampleProps} />` and see a polished checkout UI with static status

### Files (expected)

- `src/features/payment-widget/components/PaymentWidgetCard.tsx`
- `src/features/payment-widget/components/PaymentStatusTracker.tsx`
- `src/features/payment-widget/components/FeeBreakdown.tsx`
- `src/features/payment-widget/components/RoleSummary.tsx`
- `src/features/payment-widget/components/ErrorState.tsx`
- `src/features/payment-widget/components/SuccessReceipt.tsx`
- `src/features/payment-widget/components/TrustlessPaymentWidget.tsx`

---

## Issue #5 — Mock escrow mode (full offline flow)

| | |
| --- | --- |
| **Epic** | Mock |
| **Effort** | L (2–3 days) |
| **Labels** | `epic:mock`, `priority:must-have`, `size:L` |
| **Blocked by** | #3, #4 |
| **Blocks** | #6, #9 |

### Summary

Deliver a **complete mock escrow lifecycle** inside the widget so contributors and users can demo the product without API keys, wallet, or testnet USDC. Includes state machine hook, mock data store, env toggle, and wiring into `TrustlessPaymentWidget` with callbacks.

### Scope

**In scope**

- `useMockPaymentEscrow` — create → deposit → deliver → approve → release
- Mock IDs, tx hashes, timestamps
- Env flag `NEXT_PUBLIC_USE_MOCK` + widget `useMock` prop override
- Mock action buttons in widget (when mock mode; blocks hidden)
- All `PaymentWidgetCallbacks` fired at correct steps
- Initial mock demo order constants

**Out of scope**

- Real blocks/SDK (Issue #7+)
- Full demo pages (Issue #6) — but export mock data for them

### Tasks

- [ ] `useMockPaymentEscrow` — state machine aligned with `PaymentEscrowStatus`; imperative actions
- [ ] `src/lib/mock/mockEscrows.ts` — in-memory or module-level store; support multiple escrows by id
- [ ] `src/features/demo/constants/mock-order.ts` — Premium Design Audit, 500 USDC, role addresses
- [ ] Wire mock into `TrustlessPaymentWidget` when mock enabled
- [ ] Mock CTAs: Create Escrow, Deposit, Mark Delivered, Approve, Release — with loading states
- [ ] Toast notifications on step completion (sonner)
- [ ] Document mock mode in code comment + `.env.local.example`

### Acceptance criteria

- [ ] Full flow completable with `NEXT_PUBLIC_USE_MOCK=true` and no API key
- [ ] Timeline and fees update after each action
- [ ] Callbacks fire: `onEscrowCreated`, `onDepositConfirmed`, `onDelivered`, `onApproved`, `onReleased`, `onError`
- [ ] Mock tx hashes and escrow IDs generated (realistic format)
- [ ] Switching mock off shows placeholder for real mode (no crash)
- [ ] No Trustless Work API calls in mock mode

### Definition of done

- Record a 60-second screen capture of full mock flow in PR description or linked comment

### Files (expected)

- `src/features/payment-widget/hooks/useMockPaymentEscrow.ts`
- `src/lib/mock/**`
- `src/features/demo/constants/mock-order.ts`
- Updates to `TrustlessPaymentWidget.tsx`

---

## Issue #6 — Demo application (landing + four persona journeys)

| | |
| --- | --- |
| **Epic** | Demo |
| **Effort** | L (2–3 days) |
| **Labels** | `epic:demo`, `priority:must-have`, `size:L` |
| **Blocked by** | #1, #5 |
| **Blocks** | #10 |

### Summary

Build the **complete demo app** that showcases the widget in realistic commerce context: landing, shared navigation, product checkout page, and buyer/seller/admin order views. All pages use mock mode by default.

### Scope

**In scope**

- Polished landing (replace placeholder from #1)
- `DemoNav` — switch personas / routes
- `OrderSummary` shared component
- Four views: Product, Buyer, Seller, Admin
- Thin `app/` routes importing views only
- Mock order `order_design_audit_001` wired consistently

**Out of scope**

- Testnet/real mode demo (document in Issue #10)
- Evidence form polish (Issue #11)

### Tasks

- [ ] `DemoNav` — links to product page + order personas; highlight active route
- [ ] `OrderSummary` — product name, amount, status badge, seller snippet
- [ ] `ProductDemoView` — service details, trust copy, widget `mode="create"`
- [ ] `BuyerOrderView` — widget `mode="existing"`; deposit/approve/release in mock
- [ ] `SellerOrderView` — funding status, deliver action, evidence placeholder field
- [ ] `AdminOrderView` — `RoleSummary`, `FeeBreakdown`, JSON debug panel, platform-release note
- [ ] Ensure all `src/app/demo/**` pages are thin wrappers
- [ ] Responsive layouts: card list patterns on mobile where applicable

### Acceptance criteria

- [ ] Routes match `PRODUCT-BRIEF.md` §15:
  - `/demo/product/design-audit`
  - `/demo/orders/order_design_audit_001/buyer|seller|admin`
- [ ] End-to-end mock journey works across personas without page reload hacks
- [ ] Landing explains pay → hold → deliver → approve → release
- [ ] Admin page shows role mapping and fee breakdown
- [ ] No business logic duplicated in route files

### Definition of done

- New user can follow landing links and complete mock payment story across three personas in under 5 minutes

### Files (expected)

- `src/app/page.tsx`, `src/app/demo/**`
- `src/features/demo/views/**`
- `src/features/demo/components/DemoNav.tsx`, `OrderSummary.tsx`

---

## Issue #7 — Escrow Blocks integration: wallet, create, deposit & escrow fetch

| | |
| --- | --- |
| **Epic** | Blocks |
| **Effort** | XL (3–5 days) |
| **Labels** | `epic:blocks`, `priority:must-have`, `size:XL`, `help wanted` |
| **Blocked by** | #2, #3, #4, #5 |
| **Blocks** | #8, #9 |

### Summary

Wire **real testnet mode** for the first half of the escrow lifecycle: wallet connect, escrow creation via Blocks, custom fund action, indexer fetch, and escrow context for `existing`/`status` modes. This is the highest-integration issue in the MVP.

### Scope

**In scope**

- `WalletButton` in widget header
- `InitializeEscrowForm` in `mode="create"` with `buildEscrowPayload` mapping
- `FundEscrowAction` — `useFundEscrow` + sign + `useSendTransaction`
- `usePaymentEscrow` — TanStack Query + indexer hook + `mapEscrowStatus`
- `setSelectedEscrow` wiring when `escrowId` provided
- Loading/error states; toasts; trustline error messaging
- Real mode disables actions until wallet connected

**Out of scope**

- Deliver / approve / release blocks (Issue #8)
- Full orchestrator (Issue #9)

### Tasks

- [ ] Integrate `WalletButton`; gate widget actions on `isConnected`
- [ ] Create mode: embed `InitializeEscrowForm`; map widget props; single-release one milestone
- [ ] Implement `FundEscrowAction.tsx` — funder role, awaiting deposit, XDR pattern, `ErrorState` on failure
- [ ] Implement `usePaymentEscrow.ts` — query by contractId, refetch helper, map status
- [ ] Existing mode: fetch escrow on mount → `setSelectedEscrow` → render fund action if applicable
- [ ] Status mode + `readOnly`: timeline + fees only
- [ ] Handle missing/invalid escrowId with `ErrorState`
- [ ] Manual test checklist in PR: create → fund on testnet (document wallets/addresses used)

### Acceptance criteria

- [ ] With `NEXT_PUBLIC_USE_MOCK=false` and valid API key, user can create escrow on testnet
- [ ] Funder can deposit after trustline established
- [ ] `onEscrowCreated`, `onDepositStarted`, `onDepositConfirmed` callbacks fire in real mode
- [ ] Escrow fetch populates timeline correctly after fund
- [ ] Wrong network / missing trustline shows actionable error copy
- [ ] No duplicate create form — uses block, not custom form

### Definition of done

- PR includes testnet verification notes (contract ID, explorer links optional)
- At least create + fund verified by reviewer or author on dev API

### Files (expected)

- `src/features/payment-widget/components/FundEscrowAction.tsx`
- `src/features/payment-widget/hooks/usePaymentEscrow.ts`
- Updates to `TrustlessPaymentWidget.tsx`

---

## Issue #8 — Escrow Blocks integration: deliver, approve & release lifecycle

| | |
| --- | --- |
| **Epic** | Blocks |
| **Effort** | L (2–3 days) |
| **Labels** | `epic:blocks`, `priority:must-have`, `size:L` |
| **Blocked by** | #7 |
| **Blocks** | #9 |

### Summary

Complete the **on-chain lifecycle** after funding: seller marks delivered, buyer approves (with confirmation), release signer releases funds. Integrate the three remaining operation blocks with role gating and success receipt.

### Scope

**In scope**

- `ChangeMilestoneStatus`, `ApproveMilestone`, `ReleaseEscrow` blocks
- Role-based visibility (service provider, approver, release signer)
- Irreversible approve confirmation dialog
- `SuccessReceipt` after release with tx reference
- Callbacks: `onDelivered`, `onApproved`, `onReleased`

**Out of scope**

- Evidence form (Issue #11)
- Dispute UI (out of MVP)

### Tasks

- [ ] Ensure `setSelectedEscrow` set before all operation blocks
- [ ] Deliver: show for service provider when funded; status text e.g. "Delivered"
- [ ] Approve: approver only after deliver; confirm dialog; block if already approved
- [ ] Release: release signer only when approved; show `SuccessReceipt` on success
- [ ] Hide blocks when connected wallet doesn't match role (helpful copy instead)
- [ ] Refetch escrow after each successful mutation
- [ ] Testnet walkthrough: deliver → approve → release in PR notes

### Acceptance criteria

- [ ] Full testnet lifecycle completable after Issue #7 fund step
- [ ] Approve cannot proceed without explicit confirmation
- [ ] Release unavailable until milestone approved
- [ ] Timeline reflects each transition via `mapEscrowStatus`
- [ ] Platform-release role config works when `releaseSigner` is platform address

### Definition of done

- End-to-end testnet happy path documented in PR (all 5 steps)

### Files (expected)

- Updates to `TrustlessPaymentWidget.tsx` and related action shell components

---

## Issue #9 — Widget orchestration, callbacks & role-based UX

| | |
| --- | --- |
| **Epic** | Widget UI |
| **Effort** | L (2–3 days) |
| **Labels** | `epic:widget-ui`, `priority:must-have`, `size:L` |
| **Blocked by** | #5, #7, #8 |
| **Blocks** | #10 |

### Summary

Implement **`usePaymentWidget`** — the single orchestration hook that unifies mock vs real mode, derives status/fees/actions, enforces role-based UX, and fires the full callback surface. Refactor `TrustlessPaymentWidget` to be a thin presentation layer over this hook.

### Scope

**In scope**

- `usePaymentWidget(props)` — branch mock/real; expose `{ status, fees, escrow, actions, isLoading, error, isMock }`
- All callbacks from `PRODUCT-BRIEF.md` §18 including typed `onError`
- Role detection: compare wallet address to escrow roles
- Read-only mode for non-participants
- Refactor widget component to consume hook only

**Out of scope**

- New UI components
- Documentation (Issue #10)

### Tasks

- [ ] Implement `usePaymentWidget.ts` — no JSX
- [ ] Unify mock hook and real data path behind single interface
- [ ] `getAvailableActions(role, status, mode)` — centralize CTA logic
- [ ] Wire all callbacks at lifecycle points (mock + real)
- [ ] Wrong-role UX: explain why action unavailable
- [ ] Loading skeletons while fetching escrow
- [ ] Refactor `TrustlessPaymentWidget` to thin container

### Acceptance criteria

- [ ] Widget behavior identical before/after refactor (mock regression pass)
- [ ] Real mode callbacks verified for create, deposit, deliver, approve, release
- [ ] Optional callbacks don't throw when omitted
- [ ] `debug` prop still exposes role summary
- [ ] Hook is unit-test friendly (pure helpers extracted where possible)

### Definition of done

- MVP widget API stable enough to document in Issue #10 without expected breaking changes

### Files (expected)

- `src/features/payment-widget/hooks/usePaymentWidget.ts`
- Refactor `TrustlessPaymentWidget.tsx`

---

## Issue #10 — Documentation package (adoption & OSS onboarding)

| | |
| --- | --- |
| **Epic** | Docs |
| **Effort** | M (1–2 days) |
| **Labels** | `epic:docs`, `priority:must-have`, `size:M` |
| **Blocked by** | #6, #9 |
| **Blocks** | — |

### Summary

Ship the **documentation set** required for OSS adoption: component API, role mapping, demo script, and updated README. Enables developers to fork the template and embed the widget without reading the entire codebase.

### Scope

**In scope**

- `docs/component-api.md` — props, callbacks, modes, mock/real config, examples
- `docs/role-mapping.md` — buyer-release vs platform-release, security notes, trustlines
- `docs/demo-script.md` — 3-minute walkthrough; mock vs testnet steps
- README update — CLI quickstart, env vars, link to BUILD-PLAN, demo routes
- Cross-link from `PRODUCT-BRIEF.md` where helpful (index only, no rewrite)

**Out of scope**

- Extension/webhook implementation (Issue #11)

### Tasks

- [ ] Write `component-api.md` with minimal + full JSX examples
- [ ] Write `role-mapping.md` with G-address vs C-address warning
- [ ] Write `demo-script.md` for maintainers recording demos
- [ ] Update `README.md` — trustless-work init, peer deps, folder structure, mock mode
- [ ] Add docs table to README pointing to BUILD-PLAN + PRODUCT-BRIEF

### Acceptance criteria

- [ ] New developer can configure mock demo from README alone in < 15 minutes
- [ ] Component API documents all props in `PaymentWidgetProps`
- [ ] Role mapping covers default and platform-release models
- [ ] Demo script covers buyer → seller → admin flow

### Files (expected)

- `docs/component-api.md`, `docs/role-mapping.md`, `docs/demo-script.md`
- `README.md`

---

## Issue #11 — v1 polish & should-have enhancements

| | |
| --- | --- |
| **Epic** | Polish |
| **Effort** | M (1–2 days) |
| **Labels** | `priority:should-have`, `size:M` |
| **Blocked by** | #8, #9 |
| **Blocks** | — |

### Summary

Implement **should-have** items from `PRODUCT-BRIEF.md` §8: delivery evidence form, basic theming props, copyable escrow ID / tx hashes, and webhook extension notes. Optional for strict MVP but completes the v1 template experience.

### Scope

**In scope**

- `EvidenceForm` — zod + react-hook-form; URL validation; integrated with deliver flow
- `theme` prop on widget — map variant/radius to shadcn tokens where feasible
- Copy-to-clipboard for escrow ID and tx hashes
- `docs/extension-ideas.md` section on webhooks/events (docs only)

**Out of scope**

- Full webhook server implementation
- npm package publish

### Tasks

- [ ] `evidence.schema.ts` + `EvidenceForm.tsx`
- [ ] Pass evidence to deliver action / milestone metadata
- [ ] Theme prop typing + CSS variable mapping
- [ ] Clipboard UX on escrow ID and post-transaction hashes
- [ ] Document webhook integration patterns in `extension-ideas.md`

### Acceptance criteria

- [ ] Seller can submit evidence URL on seller demo page
- [ ] Theme prop changes at least radius or variant visibly
- [ ] Copy buttons work on desktop and mobile
- [ ] Extension doc explains platform backend integration at high level

### Files (expected)

- `src/features/payment-widget/schemas/evidence.schema.ts`
- `src/features/payment-widget/components/EvidenceForm.tsx`
- `docs/extension-ideas.md`
- Updates to widget props and demo seller view

---

# MVP completion checklist

| # | Criterion | Issues |
| --- | --- | --- |
| 1 | Clone and run demo locally | #1, #2, #6 |
| 2 | Mock mode without API keys | #5 |
| 3 | Clean React component API | #1, #4, #9, #10 |
| 4 | Buyer create + fund (mock) | #5, #6 |
| 5 | Seller mark delivered (mock) | #5, #6 |
| 6 | Buyer approve (mock) | #5, #6 |
| 7 | Release signer release (mock) | #5, #6 |
| 8 | Fee breakdown visible | #3, #4 |
| 9 | Role mapping in admin/debug | #4, #6 |
| 10 | Documentation for adaptation | #10 |
| 11 | TW integration isolated | #3, #7, #8 |
| 12 | Starter for checkout/marketplace/agency | #6, #10 |

**Real-mode MVP (testnet):** #7 + #8 + #9

---

# Out of scope

Per `PRODUCT-BRIEF.md` §8 and §25: multi-cart, disputes UI, refunds, fiat, inventory, accounts, messaging, notifications, multi-milestone, npm publish.

---

# Creating a GitHub issue (template)

**Before creating:** confirm phase in [Issue creation order](#issue-creation-order-for-agents--maintainers).

```markdown
## Summary
[1–2 sentences]

## BUILD-PLAN ID
Issue #N

## Effort
size:M | size:L | size:XL — [N days estimate]

## Epic
[Epic name]

## Dependencies
- Blocked by: #[GitHub issue numbers]
- Blocks: #[GitHub issue numbers]

## Scope
### In scope
- ...

### Out of scope
- ...

## Tasks
- [ ] ...

## Acceptance criteria
- [ ] ...

## Definition of done
- ...

## Files (expected)
- `paths`

## References
- docs/BUILD-PLAN.md — Issue #N
- docs/PRODUCT-BRIEF.md — §X
```

---

*Last updated: 11 consolidated issues (M/L/XL sizing), phased creation P1–P11, FOLDER_LAYOUT + Blocks compose strategy.*
