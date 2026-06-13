<p align="center"> <img src="https://github.com/user-attachments/assets/5b182044-dceb-41f5-acf0-da22dea7c98a" alt="CLR-S (2)"> </p>

# Trustless Work | [Documentation](https://docs.trustlesswork.com/trustless-work)

## Payment Widget Template

A **reusable payment widget** that lets any app accept stablecoin payments into Trustless Work escrow, holding funds until delivery, approval, or release conditions are met.

> **Add escrow-protected payments to your app in minutes.**

Built on [Trustless Work](https://docs.trustlesswork.com/trustless-work) — Escrow-as-a-Service on Stellar/Soroban — this template ships an embeddable payment component, a demo app, mock mode for UI testing, and integration docs you can fork and adapt for marketplaces, freelance platforms, e-commerce, bounties, and service deposits.

### The problem

Most payment widgets send funds **directly to the seller**. That works for low-trust-risk purchases, but not when delivery happens later, work must be reviewed, or both parties need protection before release. The missing primitive is **pay now, release later** — and Trustless Work provides that through programmable escrow.

### Core workflow

| Step | Actor | Action |
| --- | --- | --- |
| 1 | Buyer / payer | Starts payment; escrow is created for the order |
| 2 | Buyer / payer | Deposits stablecoin into escrow on Stellar |
| 3 | Seller / receiver | Sees funds secured and marks delivery or completion |
| 4 | Buyer / approver | Reviews and approves delivery |
| 5 | Release signer | Releases funds to the seller; platform fee is routed automatically |
| 6 | All parties | View payment status and timeline in the widget or demo app |

### Default role mapping (v1)

| Trustless Work role | Widget meaning | Default actor |
| --- | --- | --- |
| Payer | Buyer / client | Buyer |
| Receiver | Seller / service provider | Seller |
| Milestone Marker | Marks delivered or completed | Seller |
| Milestone Approver | Approves delivery | Buyer |
| Release Signer | Triggers fund release | Buyer (platform-controlled release optional) |
| Platform Address | Receives platform fee | Platform |

### MVP scope

**In scope (v1)**

- Embeddable payment widget with create, existing-escrow, and read-only status modes
- Single escrow payment flow: create → deposit → deliver → approve → release
- Status tracker, fee breakdown, and role mapping (payer, receiver, approver, release signer, platform)
- Mock mode (UI without real API) and real API mode (Trustless Work)
- Demo app and setup documentation

**Out of scope (future versions)**

- Multi-item cart, multi-seller checkout, full disputes, full refund flows, fiat card payments, inventory, user accounts, messaging, or notifications

For product concept, widget API, state machine, UX, and demo screens, see [`docs/PRODUCT-BRIEF.md`](docs/PRODUCT-BRIEF.md).

### Tech stack

- [Next.js 16](https://nextjs.org/) (App Router) + React 19 + TypeScript
- [Tailwind CSS v4](https://tailwindcss.com/)
- [Trustless Work SDK](https://docs.trustlesswork.com/trustless-work) — `@trustless-work/escrow` and/or `@trustless-work/blocks` (to be integrated)
- [Stellar Wallets Kit](https://github.com/Creit-Tech/Stellar-Wallets-Kit) + [Freighter](https://www.freighter.app/) (testnet for development)

---

## Local installation

### Prerequisites

- **Node.js 20+** and [pnpm](https://pnpm.io/)
- A [Trustless Work API key](https://docs.trustlesswork.com/trustless-work)
- [Freighter](https://www.freighter.app/) (or another Stellar wallet supported by Wallets Kit) on **Stellar testnet**
- Testnet USDC (or your chosen escrow asset) and a **trustline** to the asset issuer (`G…` address, not the contract `C…` address)

### Setup

```bash
# Clone the repository
git clone https://github.com/Trustless-Work/trustlesswork-payment-widget-template.git
cd trustlesswork-payment-widget-template

# Install dependencies
pnpm install

# Copy environment variables and fill in your values
cp .env.local.example .env.local   # create this file — see below
```

Create `.env.local` in the project root (never commit this file):

```env
# Trustless Work API (required once SDK/blocks are wired)
NEXT_PUBLIC_API_KEY=your_trustless_work_api_key
```

When integrating the SDK or Blocks, you will also configure role addresses, asset issuer, and provider nesting. See [Trustless Work React SDK](https://docs.trustlesswork.com/trustless-work) and the bundled agent skill at [`.agents/skills/trustless-work/`](.agents/skills/trustless-work/).

### Run locally

```bash
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000). The page hot-reloads as you edit files under `src/`.

Other scripts:

| Command | Description |
| --- | --- |
| `pnpm build` | Production build |
| `pnpm start` | Serve production build |
| `pnpm lint` | Run ESLint |

---

## Trustless Work resources & AI tools

### Platform documentation

| Resource | Link |
| --- | --- |
| Documentation home | [docs.trustlesswork.com/trustless-work](https://docs.trustlesswork.com/trustless-work) |
| REST API | [API reference](https://docs.trustlesswork.com/trustless-work) |
| React SDK (`@trustless-work/escrow`) | [SDK guides](https://docs.trustlesswork.com/trustless-work) |
| UI Blocks (`@trustless-work/blocks`) | [Blocks guides](https://docs.trustlesswork.com/trustless-work) |
| Documentation index (LLMs) | [llms.txt](https://docs.trustlesswork.com/trustless-work/llms.txt) |

**Template-specific behavior** (roles, screens, MVP boundaries) lives in this repo’s [`docs/`](docs/) folder. **Platform mechanics** (XDR signing, trustlines, API payloads) live in Trustless Work docs.

### AI Skill (Skills.sh)

Install the official Trustless Work skill so your AI assistant understands escrow lifecycle, XDR signing, trustlines, provider setup, and common integration mistakes:

```bash
npx skills add trustless-work/trustless-work-dev-skill
```

Update installed skills later:

```bash
npx skills update
```

Registry: [skills.sh/trustless-work](https://www.skills.sh/trustless-work)

This repository also ships a local copy at [`.agents/skills/trustless-work/`](.agents/skills/trustless-work/) for Cursor and compatible agents.

Learn more: [Trustless Work Skill](https://docs.trustlesswork.com/trustless-work/ai/skill)

### MCP (Cursor & compatible editors)

Connect your editor to Trustless Work documentation and escrow tools via MCP. Use **both** servers together:

- **`trustlesswork-docs`** — read docs and generate integrations
- **`trustlesswork`** — escrow actions and live operations

Add to your project `mcp.json` (project root) or Cursor **Settings → MCP**:

```json
{
  "mcpServers": {
    "trustlesswork-docs": {
      "type": "streamable-http",
      "url": "https://docs.trustlesswork.com/trustless-work/~gitbook/mcp",
      "headers": {}
    },
    "trustlesswork": {
      "type": "streamable-http",
      "url": "https://mcp.trustlesswork.com/mcp",
      "headers": {}
    }
  }
}
```

After saving, reload Cursor and confirm both servers show **Connected** under **Settings → MCP**. Use **Agent Mode** and prompts such as:

- *Create a multi-release escrow with the SDK.*
- *Generate code to call the changeMilestoneStatus endpoint.*
- *Show me how to sign a transaction for releaseFunds.*

Setup guide: [Trustless Work MCP](https://docs.trustlesswork.com/trustless-work/ai/mcp)

---

# Maintainers | [Telegram](https://t.me/+kmr8tGegxLU0NTA5)

<table align="center">
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/6b97e15f-9954-47d0-81b5-49f83bed5e4b" alt="Owner 1" width="150" />
      <br /><br />
      <strong>Tech Rebel | Product Manager</strong>
      <br /><br />
      <a href="https://github.com/techrebelgit" target="_blank">techrebelgit</a>
      <br />
      <a href="https://t.me/Tech_Rebel" target="_blank">Telegram</a>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/e245e8af-6f6f-4a0a-a37f-df132e9b4986" alt="Owner 2" width="150" />
      <br /><br />
      <strong>Joel Vargas | Frontend Developer</strong>
      <br /><br />
      <a href="https://github.com/JoelVR17" target="_blank">JoelVR17</a>
      <br />
      <a href="https://t.me/joelvr20" target="_blank">Telegram</a>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/53d65ea1-007e-40aa-b9b5-e7a10d7bea84" alt="Owner 3" width="150" />
      <br /><br />
      <strong>Armando Murillo | Full Stack Developer</strong>
      <br /><br />
      <a href="https://github.com/armandocodecr" target="_blank">armandocodecr</a>
      <br />
      <a href="https://t.me/armandocode" target="_blank">Telegram</a>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/851273f6-2f91-413d-bd2d-d8dc1f3c2d28" alt="Owner 4" width="150" />
      <br /><br />
      <strong>Caleb Loría | Smart Contract Developer</strong>
      <br /><br />
      <a href="https://github.com/zkCaleb-dev" target="_blank">zkCaleb-dev</a>
      <br />
      <a href="https://t.me/zkCaleb_dev" target="_blank">Telegram</a>
    </td>
  </tr>
</table>

---

## **Thanks to all the contributors who have made this project possible!**

[![Contributors](https://contrib.rocks/image?repo=Trustless-Work/trustlesswork-payment-widget-template)](https://github.com/Trustless-Work/trustlesswork-payment-widget-template/graphs/contributors)
