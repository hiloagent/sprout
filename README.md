# 🌱 Sprout

Solana-based tipping and crowdfunding platform. Send tips and fund projects fast, cheap, and transparently — with milestone-based escrow and NFT receipts.

## Why Solana

- Sub-cent transaction fees → micropayments actually work
- 400ms finality → instant confirmation
- Native smart contracts → trustless milestone escrow, no intermediary

## Features

### Tipping
- Tip anyone with a Solana wallet via a shareable link
- Platform fee: 1% per tip (taken at transaction time)
- Optional NFT receipt minted to tipper as proof of support

### Crowdfunding
- Create campaigns with a funding goal + deadline
- Funds held in escrow — auto-refunded if goal not reached by deadline
- Milestone-based campaigns: funds released in tranches as milestones are confirmed
- Campaign supporters receive NFT badges (early supporter, top backer, etc.)

## Architecture

```
programs/
  sprout-tip/        Anchor program — tipping with platform fee
  sprout-campaign/   Anchor program — campaign escrow + milestone vault
app/
  frontend/          Next.js + @solana/wallet-adapter
  sdk/               TypeScript SDK for interacting with programs
```

## Status

Early development. Programs being designed.

## License

MIT
