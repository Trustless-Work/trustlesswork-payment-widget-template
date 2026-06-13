# 1. Product Concept

## One-liner

A reusable payment widget that lets any app accept stablecoin payments into Trustless Work escrow, holding funds until delivery, approval, or release conditions are met.

## Simple positioning

> Add escrow-protected payments to your app in minutes.
> 

## More precise positioning

> A drop-in payment component for collecting stablecoin deposits into programmable escrow, so platforms can secure funds upfront and release them only after agreed conditions are completed.
> 

---

# 2. What It Is

The Payment Widget is a reusable frontend component plus integration template.

It allows a developer to embed a payment flow like this:

```
Buyer clicks Pay with Escrow
→ Escrow is created
→ Buyer deposits funds
→ Seller/platform sees funds secured
→ Delivery happens
→ Buyer/platform approves
→ Funds are released
```

The widget should abstract the escrow complexity behind a clean UI and developer API.

---

# 3. What It Is Not

The Payment Widget is **not**:

| Not | Why |
| --- | --- |
| A full marketplace | It should be embeddable into any marketplace |
| A full e-commerce platform | It handles payment, not catalog/cart/inventory |
| A dispute resolution platform | v1 can expose dispute status, not full arbitration |
| A fiat payment processor | Stablecoin escrow first |
| A full admin dashboard | It can include a demo dashboard, but the core product is the widget |
| A wallet product | It can integrate wallets, not replace them |

---

# 4. Target Users

## Primary user: platform developer

A developer building a product where one party pays another party, but payment should be held until something happens.

Examples:

- Marketplace developer
- Freelance platform builder
- E-commerce builder
- Service platform builder
- Bounty/grants platform builder
- Rental/deposit app builder
- Agency platform builder

## Secondary user: platform operator

The business/person that wants to add escrow payments without building escrow infrastructure.

Examples:

- Marketplace founder
- Agency owner
- Hackathon organizer
- Grant program manager
- DAO operator
- Local commerce platform
- Productized service provider

## End users

| End user | Need |
| --- | --- |
| Buyer / payer | Wants payment protection |
| Seller / receiver | Wants proof that funds are secured |
| Platform | Wants trust, fee routing, visibility, and lower operational burden |

---

# 5. Core Problem

Most payment widgets send funds directly to the seller.

That is fine for low-trust-risk purchases, but not for transactions where:

- delivery happens later
- work must be reviewed
- goods need confirmation
- milestones exist
- both parties do not fully trust each other
- the platform wants to reduce disputes
- the seller needs proof of funds before starting
- the buyer wants protection before release

The missing primitive is:

```
Pay now, release later.
```

Trustless Work can provide that primitive through escrow.

---

# 6. Core Value Proposition

## For developers

> Add escrow-backed payment flows without writing smart contracts.
> 

## For platforms

> Secure buyer funds upfront, release only when conditions are met, and monetize with platform fees.
> 

## For buyers

> Pay with protection. Funds are held until delivery is confirmed.
> 

## For sellers

> Start work knowing funds are already secured.
> 

---

# 7. Primary Use Cases

## Use Case 1: Product Purchase

A buyer pays for a physical or digital product. Funds are held until the seller marks the product as delivered and the buyer approves.

```
Buyer pays → Seller delivers → Buyer approves → Seller gets paid
```

## Use Case 2: Service Deposit

A client pays for a service. Funds are held until the service provider submits completion evidence.

```
Client deposits → Provider completes → Client approves → Provider gets paid
```

## Use Case 3: Marketplace Order

A marketplace lets a buyer pay a seller with escrow protection. The platform receives a fee at release.

```
Buyer funds escrow → Seller fulfills order → Platform/buyer approves → Seller + platform get paid
```

## Use Case 4: Booking / Reservation Deposit

A guest pays a deposit. Funds are released or refunded depending on completion conditions.

```
Guest deposits → Stay/service happens → Condition checked → Funds released/refunded
```

## Use Case 5: Bounty / Task Payment

A sponsor deposits bounty funds. Contributor gets paid after approved completion.

```
Sponsor funds → Contributor submits → Reviewer approves → Contributor gets paid
```

---

# 8. MVP Product Scope

## v1 Goal

Build a reusable payment widget that supports a **single escrow payment flow**:

```
Create escrow
→ Deposit funds
→ Mark delivered/completed
→ Approve
→ Release
```

The v1 should be simple, clean, and forkable.

## Must-Have Features

| Feature | Description |
| --- | --- |
| Embeddable payment widget | A component developers can drop into an app |
| Escrow creation | Creates a Trustless Work escrow for a payment/order |
| Deposit flow | Lets payer fund the escrow |
| Status tracker | Shows payment state from created to released |
| Delivery/completion action | Lets seller/receiver mark item or service as delivered |
| Approval action | Lets buyer/approver approve delivery |
| Release action | Releases escrowed funds after conditions are satisfied |
| Fee breakdown | Shows payment amount, platform fee, and receiver amount |
| Role mapping | Maps payer, receiver, approver, release signer, platform address |
| Mock mode | Lets developers test the UI without real API calls |
| Real API mode | Lets developers connect to Trustless Work API |
| Demo app | Shows the widget in a realistic checkout/order page |
| Documentation | Explains setup, props, flow, and customization |

## Should-Have Features

| Feature | Description |
| --- | --- |
| Delivery evidence URL | Seller can add tracking, file, proof, GitHub PR, Figma, etc. |
| Copyable escrow ID | Users can inspect or reference escrow |
| Transaction hash display | Show transaction references after important actions |
| Error state UI | Show failed transaction/API states clearly |
| Theming | Allow basic brand customization |
| Webhook/event notes | Explain how platforms can listen to escrow events |
| Demo personas | Buyer view, seller view, platform view |

## Not v1

| Excluded | Reason |
| --- | --- |
| Multi-item cart | Keep widget primitive simple |
| Multi-seller checkout | Later marketplace template |
| Full disputes | Too much legal/product complexity |
| Full refund flow | Can be v2 after release/cancel flows are clear |
| Fiat card payment | Separate on/off-ramp integration |
| Inventory system | Not core to escrow payment |
| User accounts | Demo can use wallet/address-based identity |
| Messaging | Not core to payment primitive |
| Notifications | Can be documented as extension |

---

# 9. Widget Modes

The widget should support **three modes**.

## Mode 1: Payment Creation Mode

Used when no escrow exists yet.

```tsx
<TrustlessPaymentWidget
  mode="create"
  amount={500}
  currency="USDC"
  payerAddress="G..."
  receiverAddress="G..."
  platformAddress="G..."
  platformFeeBps={250}
/>
```

Primary action:

```
Create Escrow
```

Then:

```
Deposit Funds
```

## Mode 2: Existing Escrow Mode

Used when the platform already created the escrow and wants the widget to show status/actions.

```tsx
<TrustlessPaymentWidget
  mode="existing"
  escrowId="escrow_123"
/>
```

Primary behavior:

```
Fetch escrow status
→ Render next available action
```

## Mode 3: Read-Only Status Mode

Used for order pages, receipts, dashboards, public escrow viewers, or customer support.

```tsx
<TrustlessPaymentWidget
  mode="status"
  escrowId="escrow_123"
  readOnly
/>
```

Primary behavior:

```
Show escrow state only
```

---

# 10. Widget Props

## Minimal API

```tsx
<TrustlessPaymentWidget
  amount={500}
  currency="USDC"
  payerAddress="G..."
  receiverAddress="G..."
  approverAddress="G..."
  releaseSignerAddress="G..."
  platformAddress="G..."
  platformFeeBps={250}
  title="Premium Design Audit"
  description="Escrow-protected payment for a UX audit."
/>
```

## Full API

```tsx
<TrustlessPaymentWidget
  mode="create"
  network="testnet"
  amount={500}
  currency="USDC"
  assetCode="USDC"
  title="Premium Design Audit"
  description="Escrow-protected payment for a UX audit."
  externalReference="order_123"

  payerAddress="G..."
  receiverAddress="G..."
  milestoneMarkerAddress="G..."
  approverAddress="G..."
  releaseSignerAddress="G..."
  platformAddress="G..."
  platformFeeBps={250}

  trustlessWorkFeeBps={30}

  metadata={{
    productId: "prod_123",
    orderId: "order_123",
    sellerId: "seller_456"
  }}

  theme={{
    variant: "default",
    radius: "lg"
  }}

  callbacks={{
    onEscrowCreated: (escrow) => {},
    onDepositStarted: (escrowId) => {},
    onDepositConfirmed: (tx) => {},
    onDelivered: (escrowId) => {},
    onApproved: (escrowId) => {},
    onReleased: (tx) => {},
    onError: (error) => {}
  }}
/>
```

---

# 11. Core State Machine

The widget should be built around a clear payment state machine.

```
idle
→ creating_escrow
→ awaiting_deposit
→ funded
→ awaiting_delivery
→ delivered
→ awaiting_approval
→ approved
→ ready_to_release
→ releasing
→ released
```

Additional states:

```
cancelled
disputed
expired
error
```

## User-facing status labels

| Internal State | User Label |
| --- | --- |
| `idle` | Ready to start |
| `creating_escrow` | Creating escrow |
| `awaiting_deposit` | Awaiting payment |
| `funded` | Funds secured |
| `awaiting_delivery` | Awaiting delivery |
| `delivered` | Delivery submitted |
| `awaiting_approval` | Awaiting approval |
| `approved` | Approved |
| `ready_to_release` | Ready to release |
| `releasing` | Releasing payment |
| `released` | Payment released |
| `disputed` | Issue raised |
| `cancelled` | Cancelled |
| `error` | Action failed |

---

# 12. Role Mapping

For v1, use a simple single-payment escrow.

| Trustless Work Role | Payment Widget Meaning |
| --- | --- |
| Payer | Buyer / client |
| Receiver | Seller / service provider |
| Milestone Marker | Seller marks delivered/completed |
| Milestone Approver | Buyer approves delivery |
| Release Signer | Buyer, platform, or configured signer |
| Platform Address | Receives platform fee |
| Admin | Optional platform admin |
| Dispute Resolver | Not active in v1 |

## Recommended default role mapping

```tsx
const defaultRoles = {
  payer: buyerAddress,
  receiver: sellerAddress,
  milestoneMarker: sellerAddress,
  milestoneApprover: buyerAddress,
  releaseSigner: buyerAddress,
  platformAddress: platformAddress
};
```

## Alternative platform-controlled release model

```tsx
const platformReleaseRoles = {
  payer: buyerAddress,
  receiver: sellerAddress,
  milestoneMarker: sellerAddress,
  milestoneApprover: buyerAddress,
  releaseSigner: platformAddress,
  platformAddress: platformAddress
};
```

Recommendation for v1:

> Support both, but make buyer-release the default because it is simpler to understand.
> 

---

# 13. UX Requirements

## Primary widget layout

The widget should feel like a modern payment component, not a technical contract interface.

```
┌─────────────────────────────────────┐
│ Pay with Escrow                     │
│ Premium Design Audit                │
│ $500 USDC                           │
│                                     │
│ Funds are held securely until       │
│ delivery is approved.               │
│                                     │
│ Status: Awaiting payment            │
│                                     │
│ [ Create Escrow ]                   │
└─────────────────────────────────────┘
```

After escrow creation:

```
┌─────────────────────────────────────┐
│ Escrow Payment                      │
│ $500 USDC                           │
│                                     │
│ Status: Awaiting deposit            │
│ Escrow ID: esc_123                  │
│                                     │
│ [ Deposit Funds ]                   │
└─────────────────────────────────────┘
```

After funded:

```
┌─────────────────────────────────────┐
│ Funds Secured                       │
│ $500 USDC held in escrow            │
│                                     │
│ Waiting for seller to deliver.      │
│                                     │
│ Timeline:                           │
│ ✓ Escrow created                    │
│ ✓ Funds deposited                   │
│ ○ Delivery submitted                │
│ ○ Buyer approved                    │
│ ○ Payment released                  │
└─────────────────────────────────────┘
```

---

# 14. User Views

The widget should render different actions depending on the connected role.

## Buyer View

| State | Buyer Action |
| --- | --- |
| No escrow | Create escrow / start payment |
| Awaiting deposit | Deposit funds |
| Delivered | Approve delivery |
| Approved | Release payment, if buyer is release signer |
| Released | View receipt |
| Disputed | View issue state |

## Seller View

| State | Seller Action |
| --- | --- |
| Awaiting deposit | Waiting for buyer payment |
| Funded | Mark as delivered |
| Delivered | Waiting for buyer approval |
| Released | View payment receipt |

## Platform View

| State | Platform Action |
| --- | --- |
| Any | View escrow status |
| Approved | Release payment, if platform is release signer |
| Disputed | View issue state |
| Released | View fee receipt |

---

# 15. Screens for Demo App

The repo should include a demo app to show the widget in context.

## Demo Page 1: Product Page

```
/product/design-audit
```

Shows:

- Product/service details
- Seller information
- Price
- Trustless Work Payment Widget

## Demo Page 2: Buyer Order Page

```
/orders/order_123/buyer
```

Shows:

- Order summary
- Escrow status
- Buyer actions
- Evidence submitted by seller

## Demo Page 3: Seller Order Page

```
/orders/order_123/seller
```

Shows:

- Order summary
- Funding status
- Seller delivery action
- Delivery evidence form

## Demo Page 4: Platform Admin Page

```
/admin/orders/order_123
```

Shows:

- Full role mapping
- Escrow status
- Fee breakdown
- Release status
- Debug/API payload preview

---

# 16. Fee Model Display

The widget should make fees visible.

Example:

```
Payment amount:        500.00 USDC
Platform fee:           12.50 USDC
Trustless Work fee:      1.50 USDC
Seller receives:       486.00 USDC
```

Assuming:

```
Platform fee = 2.5%
Trustless Work fee = 0.3%
```

The widget should accept:

```tsx
platformFeeBps={250}
trustlessWorkFeeBps={30}
```

Basis points are important because later this maps cleanly to smart contract fee logic.

---

# 17. API Integration Layer

The frontend should not call scattered API functions directly from components.

Create a clean service layer:

```
lib/trustless-work/
  createEscrow.ts
  getEscrow.ts
  depositFunds.ts
  markDelivered.ts
  approveDelivery.ts
  releasePayment.ts
  mapEscrowStatus.ts
```

## Example function shape

```tsx
export async function createPaymentEscrow(input: CreatePaymentEscrowInput) {
  // 1. Validate input
  // 2. Map widget props to Trustless Work escrow payload
  // 3. Call Trustless Work API
  // 4. Return normalized escrow object
}
```

---

# 18. Suggested Type Definitions

## Payment Widget Props

```tsx
export type PaymentWidgetProps = {
  mode?: "create" | "existing" | "status";
  network?: "testnet" | "mainnet";

  amount?: number;
  currency?: "USDC";
  title?: string;
  description?: string;
  externalReference?: string;

  payerAddress?: string;
  receiverAddress?: string;
  milestoneMarkerAddress?: string;
  approverAddress?: string;
  releaseSignerAddress?: string;
  platformAddress?: string;

  platformFeeBps?: number;
  trustlessWorkFeeBps?: number;

  escrowId?: string;
  readOnly?: boolean;

  metadata?: Record<string, string>;

  callbacks?: PaymentWidgetCallbacks;
};
```

## Payment Widget Callbacks

```tsx
export type PaymentWidgetCallbacks = {
  onEscrowCreated?: (escrow: PaymentEscrow) => void;
  onDepositStarted?: (escrowId: string) => void;
  onDepositConfirmed?: (txHash: string) => void;
  onDelivered?: (escrowId: string) => void;
  onApproved?: (escrowId: string) => void;
  onReleased?: (txHash: string) => void;
  onError?: (error: PaymentWidgetError) => void;
};
```

## Payment Escrow

```tsx
export type PaymentEscrow = {
  escrowId: string;
  externalReference?: string;
  title: string;
  description?: string;
  amount: number;
  currency: "USDC";
  status: PaymentEscrowStatus;
  roles: PaymentEscrowRoles;
  fees: PaymentEscrowFees;
  evidence?: PaymentEvidence;
  createdAt: string;
  updatedAt: string;
};
```

## Status

```tsx
export type PaymentEscrowStatus =
  | "idle"
  | "creating_escrow"
  | "awaiting_deposit"
  | "funded"
  | "awaiting_delivery"
  | "delivered"
  | "awaiting_approval"
  | "approved"
  | "ready_to_release"
  | "releasing"
  | "released"
  | "cancelled"
  | "disputed"
  | "expired"
  | "error";
```

---

# 19. Repository Structure

```
trustless-work-payment-widget/
  app/
    page.tsx
    demo/
      product/
        page.tsx
      buyer/
        page.tsx
      seller/
        page.tsx
      admin/
        page.tsx

  components/
    payment-widget/
      TrustlessPaymentWidget.tsx
      PaymentWidgetCard.tsx
      PaymentStatusTracker.tsx
      PaymentActionButton.tsx
      FeeBreakdown.tsx
      RoleSummary.tsx
      EvidenceForm.tsx
      ErrorState.tsx
      SuccessReceipt.tsx

  lib/
    trustless-work/
      createEscrow.ts
      getEscrow.ts
      depositFunds.ts
      markDelivered.ts
      approveDelivery.ts
      releasePayment.ts
      mapEscrowStatus.ts

    mock/
      mockEscrows.ts
      mockOrders.ts
      mockUsers.ts

    utils/
      formatCurrency.ts
      shortenAddress.ts
      calculateFees.ts
      assertPaymentWidgetInput.ts

  types/
    payment-widget.ts
    escrow.ts
    roles.ts
    fees.ts

  docs/
    product-definition.md
    component-api.md
    user-flows.md
    role-mapping.md
    state-machine.md
    api-integration.md
    demo-script.md
    extension-ideas.md

  README.md
  .env.example
  package.json
```

---

# 20. README Structure

```markdown
# Trustless Work Payment Widget

Add escrow-protected stablecoin payments to your app.

## What this is

A reusable payment widget for creating and managing Trustless Work escrows.

## Use cases

- Product checkout
- Service deposits
- Marketplace orders
- Bounties
- Booking deposits
- Milestone payments

## Quickstart

1. Clone repo
2. Install dependencies
3. Configure env
4. Run demo
5. Add widget to your app

## Example

<TrustlessPaymentWidget ... />

## Escrow flow

Pay → Hold → Deliver → Approve → Release

## Role mapping

Buyer = payer  
Seller = receiver  
Seller = milestone marker  
Buyer = approver  
Buyer/platform = release signer  
Platform = fee recipient

## Demo modes

- Mock mode
- Testnet mode
- Mainnet-ready config

## Extending this template

- Add disputes
- Add refunds
- Add notifications
- Add marketplace orders
- Add embedded wallets
```

---

# 21. Open-Source Issues / Task Breakdown

## Epic 1: Project Setup

### Issue 1 — Initialize Next.js + TypeScript template

Build base project with:

- Next.js
- TypeScript
- Tailwind
- basic layout
- demo routes

Acceptance criteria:

- App runs locally
- Routes exist for product, buyer, seller, admin demos
- Basic layout is responsive

### Issue 2 — Define payment widget types

Create shared types for:

- widget props
- escrow status
- roles
- fees
- callbacks
- evidence

Acceptance criteria:

- Types are exported from `/types`
- Components consume shared types
- No duplicated status strings across components

## Epic 2: Core Widget UI

### Issue 3 — Build `TrustlessPaymentWidget` shell

Acceptance criteria:

- Accepts minimal props
- Renders title, amount, description
- Shows current status
- Supports mock mode
- Supports read-only mode

### Issue 4 — Build payment status tracker

Create timeline component:

```
Created → Funded → Delivered → Approved → Released
```

Acceptance criteria:

- Shows active state
- Shows completed states
- Handles error/disputed/cancelled states
- Works on mobile

### Issue 5 — Build fee breakdown component

Create `FeeBreakdown`.

Acceptance criteria:

- Accepts amount, platformFeeBps, trustlessWorkFeeBps
- Calculates platform fee
- Calculates Trustless Work fee
- Calculates receiver amount
- Handles zero platform fee

### Issue 6 — Build role summary component

Create role display for:

- payer
- receiver
- marker
- approver
- release signer
- platform address

Acceptance criteria:

- Shows shortened addresses
- Explains role labels in simple language
- Works in admin/debug mode

## Epic 3: Mock Escrow Flow

### Issue 7 — Create mock escrow state machine

Build local mock flow for:

```
create → deposit → deliver → approve → release
```

Acceptance criteria:

- User can run full flow without API keys
- State updates correctly
- Status tracker updates after each action
- Mock transaction IDs are generated

### Issue 8 — Build buyer demo page

Acceptance criteria:

- Buyer can create escrow
- Buyer can deposit
- Buyer can approve after delivery
- Buyer can release if configured as release signer

### Issue 9 — Build seller demo page

Acceptance criteria:

- Seller can view funded escrow
- Seller can mark delivered
- Seller can submit evidence URL
- Seller can see when payment is released

### Issue 10 — Build admin demo page

Acceptance criteria:

- Admin can view full escrow object
- Admin can view role mapping
- Admin can view fee breakdown
- Admin can trigger release if platform release mode is enabled

## Epic 4: Trustless Work API Integration

### Issue 11 — Create Trustless Work API client wrapper

Acceptance criteria:

- Reads API key from env
- Supports testnet/mainnet config
- Handles API errors
- Exposes normalized methods

### Issue 12 — Implement `createPaymentEscrow`

Acceptance criteria:

- Maps widget props to Trustless Work escrow payload
- Supports single-payment escrow
- Supports platform fee
- Returns normalized escrow object

### Issue 13 — Implement escrow status fetch

Acceptance criteria:

- Fetches escrow by ID
- Maps API response to widget state
- Handles missing/invalid escrow
- Supports polling or manual refresh

### Issue 14 — Implement milestone delivery update

Acceptance criteria:

- Seller/marker can mark delivered
- Evidence URL can be attached or stored in metadata if supported
- UI updates after successful action

### Issue 15 — Implement approval action

Acceptance criteria:

- Approver can approve delivery/milestone
- UI updates to approved state
- Release action becomes available only after approval

### Issue 16 — Implement release action

Acceptance criteria:

- Release signer can release funds
- Release only available after approval
- Shows transaction result
- Shows final receipt

## Epic 5: Documentation

### Issue 17 — Write product definition doc

Include:

- Product purpose
- Target users
- Use cases
- v1 scope
- non-goals

### Issue 18 — Write component API docs

Include:

- Minimal usage
- Full usage
- Props
- Callbacks
- Modes
- Theming
- Common examples

### Issue 19 — Write role mapping docs

Include:

- Default role mapping
- Buyer-release model
- Platform-release model
- Security considerations

### Issue 20 — Write demo script

Include a 3-minute flow:

```
Buyer creates escrow
Buyer deposits
Seller delivers
Buyer approves
Payment releases
```

---

# 22. Design Principles

## Keep the widget simple

The widget should hide complexity by default.

Developer can enable advanced/debug mode if needed.

```tsx
<TrustlessPaymentWidget debug />
```

## Make escrow understandable

Avoid blockchain-heavy copy.

Use:

```
Funds secured
Awaiting delivery
Delivery submitted
Approved
Payment released
```

Avoid:

```
Contract state changed
Milestone mutation successful
Signer authorization completed
```

## Optimize for copy-paste adoption

The repo should make developers think:

> I can use this in my app today.
> 

## Support real business flows

The widget should feel useful for actual commerce, not only a hackathon demo.

---

# 23. UX Copy

## Main headline options

```
Pay with Escrow
```

```
Secure Payment with Escrow
```

```
Deposit Funds Securely
```

```
Pay Now, Release After Delivery
```

Recommendation:

```
Pay with Escrow
```

## Helper copy

```
Your payment is held securely until delivery is approved.
```

## Seller-side copy

```
Funds are secured. You can now deliver the order.
```

## Approval copy

```
Review the delivery and approve payment release.
```

## Release copy

```
Payment is ready to be released to the seller.
```

## Final receipt copy

```
Payment released successfully.
```

---

# 24. MVP Acceptance Criteria

The MVP is complete when:

1. A developer can clone the repo and run the demo locally.
2. The widget can run in mock mode without API keys.
3. The widget exposes a clean React component API.
4. A buyer can create/fund a mock escrow.
5. A seller can mark delivery as complete.
6. A buyer can approve delivery.
7. A release signer can release payment.
8. Fee breakdown is visible.
9. Role mapping is visible in admin/debug mode.
10. Documentation explains how to adapt the widget to a real app.
11. Trustless Work API integration points are clearly isolated.
12. The repo can be used as a starter for checkout, marketplace, agency, bounty, or deposit flows.

---

# 25. v2 Ideas

| v2 Feature | Why |
| --- | --- |
| Refund flow | Needed for failed delivery |
| Dispute flow | Needed for marketplaces |
| Partial release | Needed for milestones |
| Multi-release escrow | Needed for agencies and grants |
| Embedded wallet support | Better UX for non-crypto users |
| On-ramp support | Let users fund with card/bank |
| Email notifications | Buyer/seller updates |
| Webhook listener example | Platform backend integration |
| Package export | Publish as npm package |
| Theme system | Better integration into partner apps |
| Multi-chain support | Future chain expansion |

---

# 26. Strategic Recommendation

Build the Payment Widget as the **base primitive** for all future templates.

Then reuse it inside:

```
Agency Template
Marketplace Template
Bounty Template
Security Deposit Template
Agentic Work Template
```

This means the widget should be built as a real component, not just as a one-off demo.

## Best repo framing

```
trustless-work-payment-widget
```

## Best first demo

Use a simple **service/product checkout**:

```
Premium Design Audit — 500 USDC
```

Why this is better than a physical product:

- No inventory complexity
- No shipping complexity
- Easy to understand delivery evidence
- Maps well to agencies and freelancers
- Good bridge to the agency template

## Best first flow

```
Client pays 500 USDC into escrow
Provider submits delivery evidence
Client approves
Payment releases to provider
Platform fee is deducted
```

This gives us the cleanest possible demo of Trustless Work as an embeddable escrow payment layer.