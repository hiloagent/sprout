# Sprout — Design Document

## Programs

### sprout-tip

Simple tipping with platform fee.

**Accounts:**
- `TipConfig` — global PDA: platform fee %, treasury wallet
- `TipReceipt` — per-tip PDA: tipper, recipient, amount, timestamp, optional message

**Instructions:**
- `initialize(fee_bps: u16, treasury: Pubkey)` — one-time setup
- `tip(amount: u64, message: Option<String>)` — transfer SOL to recipient, deduct fee to treasury, create receipt
- `tip_with_nft(amount, message)` — same + mint NFT receipt to tipper

**Fee model:**
- Default: 100 bps (1%)
- Fee taken from tip amount: recipient gets `amount * (1 - fee_bps/10000)`

---

### sprout-campaign

Crowdfunding with goal-based escrow and milestone tranches.

**Campaign types:**
- `AllOrNothing` — full refund if goal not reached by deadline
- `Milestone` — funds released in tranches as milestones are confirmed by campaign owner

**Accounts:**
- `Campaign` — PDA: owner, title, goal_amount, deadline, campaign_type, status, raised_amount
- `Milestone` — PDA per milestone: description, amount_bps (% of goal), completed, released
- `Pledge` — PDA per backer: campaign, backer, amount, refunded
- `CampaignVault` — escrow token account holding funds

**Instructions:**
- `create_campaign(title, goal_amount, deadline, campaign_type, milestones[])` — initialise campaign + vault
- `pledge(amount)` — transfer SOL to vault, create Pledge account
- `complete_milestone(milestone_index)` — owner marks milestone done, release tranche to owner
- `finalize_campaign()` — after deadline: if AllOrNothing + goal met → release all; if not met → enable refunds
- `claim_refund()` — backer claims refund if campaign failed
- `cancel_campaign()` — owner cancels before deadline, enables refunds

**NFT badges (Metaplex):**
- Early supporter (first 10% of backers)
- Top backer (top 5 by amount)
- Milestone achiever (campaign completed all milestones)

---

## Frontend pages

- `/` — discover campaigns, featured creators
- `/tip/[wallet]` — tip page for a wallet address
- `/campaign/[id]` — campaign page: progress, milestones, backers
- `/create` — create campaign wizard
- `/dashboard` — creator dashboard: campaigns, tips received, treasury

## SDK

```typescript
import { SproutClient } from '@sprout/sdk';

const client = new SproutClient({ connection, wallet });

// tipping
await client.tip({ recipient, amount: 0.1, message: 'Great work!' });

// campaigns
const campaign = await client.createCampaign({
  title: 'My Project',
  goalAmount: 100, // SOL
  deadline: new Date('2026-06-01'),
  type: 'milestone',
  milestones: [
    { description: 'MVP launched', amountBps: 3000 },
    { description: 'Beta users', amountBps: 3000 },
    { description: 'v1.0 shipped', amountBps: 4000 },
  ]
});

await client.pledge({ campaign: campaign.id, amount: 1 });
```

## Fee strategy

- Tipping: 1% of tip amount
- Campaign creation: free
- Successful campaign payout: 2% of raised amount
- Failed/refunded campaign: no fee

All fees go to platform treasury wallet. Configurable via TipConfig PDA (owner-only).
