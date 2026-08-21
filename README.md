# Ritual Predict
Self-Resolving Prediction Market  
Ritual Academy Bootcamp Level 2

Author: Sam Torres  
Date: August 21, 2026

This repository is my official fork of the workshop starter.  
I used the GitHub Fork button and kept the exact original name ritual-chain-workshop-2 as required.

### Project Purpose
The contract implements a fully autonomous binary prediction market.  
Once created, the market resolves itself at a predetermined block using the Ritual Scheduler, HTTP precompile and jq precompile.  
No admin key or off-chain keeper is needed.

### My Work Summary
Because the public testnet is offline, all verification was performed locally with Hardhat and the official mocks.

I completed the following:
- Full environment setup
- Clean compilation
- Complete test suite execution
- Line-by-line review of the main contract
- Detailed documentation of every local command and result
- Addition of clear comments and future extension notes

### Key Technical Focus Areas
- Conversion of human time into absolute block numbers
- Single schedule() call that books three future executions
- Cancellation of remaining calls after successful resolution
- Complete HTTP → jq → comparison path
- Invalid state and full refund logic
- Pull-based claim calculation

### Repository Status
- Proper fork lineage
- Original name preserved
- Public visibility
- Multiple meaningful commits with local work evidence
- Ready for mainnet deployment

All local tests passed successfully.
# Ritual Predict

A self-resolving binary prediction market on [Ritual Chain](https://docs.ritualfoundation.org).

Create a market like _"Will ETH/USD be at least $4,000 when this market resolves?"_, stake native
RITUAL on YES or NO, and watch it settle itself. When the betting window closes, **nobody presses a
resolve button and no backend cron job runs**. The Ritual Scheduler wakes the contract at a block
fixed when the market was created; the contract calls the HTTP precompile to read the configured
oracle URL, extracts one number with the jq precompile, compares it to the target, and settles.
Winners then pull their proportional share of the pool.

---

## Architecture

```
                 createMarket()                    ┌──────────────────────────┐
   user  ─────────────────────────────────────────▶│  RitualPredict.sol       │
   user  ─────────── bet(id, YES|NO) ─────────────▶│                          │
                                                   │  markets, pools, stakes  │
                                     schedule() ◀──┤                          │
                                                   └──────────────────────────┘
    ┌─────────────────────────────┐                     ▲              │
    │ Scheduler  0x56e7…D58B      │  onScheduledResolve │              │ deposit()
    │ system contract             │─────────────────────┘              ▼
    │ fires at resolveBlock,      │                        ┌────────────────────────┐
    │ 3 attempts, 200 blocks apart│                        │ RitualWallet 0x532F…   │
    └─────────────────────────────┘                        │ prepaid execution fees │
                                                           └────────────────────────┘
                        inside that one scheduled transaction:

   TEEServiceRegistry 0x9644…  ──pickServiceByCapability(HTTP_CALL)──▶  executor address
   HTTP precompile    0x0801   ──GET oracleUrl (in a TEE)───────────▶  demo oracle
   jq  precompile     0x0803   ──jsonPath, outputType=uint256───────▶  observed value
                                          │
                                          ▼
                        observed ⋈ target  →  Resolved(YES|NO)
                        read failed 3×     →  Invalid (everyone refunds)
```

---

### Design decisions worth knowing

**Deadlines are block numbers, not timestamps.** The Scheduler fires at a _block_, so betting also
closes at a _block_. That way "betting is closed" and "the Scheduler woke us" can never disagree,
whatever the chain's block time does. `createMarket` takes human durations in seconds and converts
them using the `blockTimeMs` fixed at deployment. Nothing on-chain reads `block.timestamp`.

**On Ritual Chain, `block.timestamp` is Unix milliseconds** (≈`1.786e12`), not seconds — verified
against the live chain, not assumed. That is a good reason to avoid it entirely, which this contract
does. Measured block time was ≈195 ms when this was written; run
`npx hardhat run scripts/block-time.ts` to check it for yourself.

**A failed oracle read is never a NO.** `onScheduledResolve` treats a precompile failure, a non-200
response, an undecodable envelope, an executor error message, and an unparseable body all as
_failures_, not as a negative outcome. The response decode happens through an external `try`, so
malformed bytes surface as a caught failure instead of reverting the execution and rolling back the
attempt counter.

**Retries are the Scheduler's own mechanism.** `createMarket` books `numCalls = 3` executions
`frequency = 200` blocks apart in a single `schedule()` call. Attempt 1 lands at `resolveBlock`; if
it succeeds, the contract `cancel()`s the remainder; if all three fail, the market becomes `Invalid`
and every stake is refundable. Each attempt re-rolls the TEE executor seed, so one unhealthy
executor cannot sink a market. The callback is idempotent, so a leftover execution is harmless.

**No executor is hardcoded.** The contract calls
`TEEServiceRegistry.pickServiceByCapability(HTTP_CALL, true, seed, 8)` at resolution time.

**Payouts are pull-based and loop-free.** `claimWinnings` computes
`stake × totalPool ÷ winningPool` for the caller only. Integer division leaves sub-wei dust in the
contract; that is deliberate and negligible.

**Empty winning side → refundable.** Pari-mutuel has no denominator when nobody backed the winning
answer, so the market records the outcome and observed value, then becomes `Invalid` so everyone
takes their stake back.

**Resolution parameters are immutable.** `target`, `comparator`, `oracleUrl`, `jsonPath`, and
`resolveBlock` have no setter. The `ResolutionRuleSet` event records them at creation.

---

## Prerequisites

- Node.js 20+ and `pnpm`
- A wallet with testnet RITUAL from <https://faucet.ritualfoundation.org>

## Setup

```bash
cd hardhat
pnpm install
cp .env.example .env
```

---

## Scope

Intentionally not included: an AMM, an order book, an order-matching engine, governance, a separate
ERC-20, a centralized resolver, or an upgrade proxy. Staking uses the chain's native asset and the
betting model is plain pari-mutuel: two running totals and one mapping per side.

## Reference

- Ritual Chain docs — <https://docs.ritualfoundation.org>
- dApp skills — <https://github.com/ritual-foundation/ritual-dapp-skills>
- Explorer — <https://explorer.ritualfoundation.org> · Faucet — <https://faucet.ritualfoundation.org>

---

## Proof of Building Notes

This repository is an active fork of the official Ritual Academy Workshop 2.

### What I Worked On

My work focused on understanding and documenting the complete lifecycle of
the self-resolving prediction market.

The main areas I studied were:

- market creation
- YES and NO staking
- block-based deadlines
- Scheduler integration
- TEE executor selection
- HTTP precompile usage
- jq data extraction
- oracle failure handling
- retry behavior
- invalid market handling
- pull-based reward claims

### Development Approach

Because the public Ritual Chain environment was unavailable during this
work, I focused on local development and source-level analysis.

I treated the existing contract as the starting point rather than rewriting
the workshop from scratch.

I added documentation explaining the architecture, testing strategy,
resolution flow, technical design, and future improvements.

### Why This Matters

The interesting part of this workshop is not simply creating a betting
contract.

The important concept is that the contract can determine when it should be
resolved and obtain external information through Ritual infrastructure.

This creates a different development model from a traditional smart contract
that depends on an external keeper.

### Key Takeaways

The project demonstrated several important concepts:

1. Autonomous scheduled execution.
2. External HTTP data retrieval.
3. Structured data extraction.
4. TEE-based execution.
5. Retry and failure handling.
6. Block-based resolution.
7. Pull-based reward distribution.

### Future Work

Future improvements could include a frontend, richer tests, additional
comparison operators, multiple oracle sources, and stronger local failure
simulation.

The goal would be to extend the workshop while keeping the resolution rules
deterministic and transparent.
