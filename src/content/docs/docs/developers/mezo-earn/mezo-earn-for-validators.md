---
title: Mezo Earn for Validators
description: >-
  How validators participate in Mezo's economic engine, receive incentives, and
  interact with veBTC, veMEZO, gauges, and LP staking.
topic: developers
---

This page explains how validators participate in Mezo's economic engine (Mezo Earn), how they receive incentives, and how this ties into veBTC, veMEZO, gauges, and LP staking.

It is written for:

- Current and prospective Mezo validators
- Infrastructure providers and partners
- veBTC/veMEZO holders and LPs who want to understand how validator rewards work

:::note

The Mezo whitepaper is the source of truth for the mechanics and formulas behind veBTC, veMEZO, gauges, splitters, and emissions.

:::

---

## High-Level Overview

At a high level:

- Validators secure the Mezo chain and earn MEZO rewards routed through validator gauges in Mezo Earn.
- veBTC holders are the governance anchor. They vote on gauges to direct emissions within each branch of the splitter system.

**Key Mechanics:**

- **Emission Splits:** Splitter ratios (Chain Splitter and Ecosystem Splitter) are currently fixed and locked via governance. They may become adjustable through future governance mechanisms.
- **veMEZO Role:** veMEZO acts as a boost on veBTC power (up to a capped multiplier) but never replaces BTC as the anchor.

---

## Core Concepts

### Key Definitions

To understand Mezo Earn, we distinguish between three economic terms:

- **Rewards:** What actors receive in exchange for their work (e.g., MEZO emissions).
- **Incentives:** The rules or benefits designed to encourage specific behaviors (e.g., earning a boost for holding veMEZO).
- **Bribes:** A temporary budget provided by protocols or users to promote a specific action (e.g., voting for a specific gauge).

### Actors in Mezo Earn

Mezo Earn coordinates five primary actors:

1. **End-users / veBTC & veMEZO lockers:** Lock BTC or MEZO, receive veBTC and veMEZO NFTs, vote in gauges, and claim fees/emissions.
2. **Liquidity Providers (LPs):** Provide DEX liquidity and receive LP tokens and fees; they can optionally stake LP tokens in staking gauges to trade fees for MEZO emissions.
3. **Validators:** Run Mezo chain nodes, participate in consensus, and receive MEZO emissions through validator gauges.
4. **Protocols:** Deploy bribes on gauges to attract liquidity or governance votes.
5. **The Chain:** Collects swap and chain/bridging fees and redistributes them through Earn.

---

## Validator Gauges and veBTC Voting

Validators participate in Mezo Earn through validator gauges.

:::note

For an initial period before validator voting is available, Mezo will use its own veBTC to vote equally between the validators.

:::

### How Validator Gauges Work

The chain splitter sends a fixed share of reward emissions to the validator branch, currently set at 20% of reward emissions. This ratio is locked via governance.

Within the validator branch:

- Each validator has a non-staking gauge.
- veBTC + veMEZO votes on that gauge determine its share of the 20% validator emissions bucket.

Emissions received by a validator gauge can be:

- Kept by the validator operator.
- Shared with veBTC voters (delegator-like behavior) by reposting some rewards as incentives on their own gauge.
- Combined with off-chain co-incentives (paid by the validator entity).

### Voting for Validators with veBTC and veMEZO

- **Who votes:** veBTC holders with optional veMEZO boost.
- **Effect of a vote:** Allocates voting power to specific validator gauges.
- **Influences:**
  - The distribution of MEZO emissions to validators (within the validator branch).
  - Indirectly, the economic attractiveness of each validator to veBTC delegators.

#### Conceptual Example

- Suppose the validator branch receives 20% of the weekly reward emissions via the chain splitter.
- If a single validator gauge receives 25% of validator voting weight, that validator receives 25% of the validator-branch emissions for that epoch (meaning 5% of the total weekly emissions, since 25% of 20% is 5%).
- If that validator reposts part of those emissions back as incentives on its own gauge, veBTC voters supporting it can share in the upside on future epochs.

---

## Step-by-Step: Voting for Validator Gauges (UI Path)

1. **Connect wallet to the Mezo Earn interface:** Connect a wallet holding veBTC (and optionally veMEZO).

2. **Navigate to the Validators section:** View the list of validator gauges, including name/operator, past performance/uptime, and current emissions share and incentives.

3. **Allocate votes:**
   - Select a validator.
   - Choose the percentage of your available virtual voting weight to allocate during that epoch.
   - Confirm and sign the transaction.

4. **Confirm on explorer:** Use the Mezo block explorer to verify vote transaction success and the updated gauge weight.

5. **Claim rewards:** At or after epoch end, claim your share of emissions/incentives attached to the validator gauge (if reposted) and any additional rewards.

:::note

Exact UI implementations may evolve. The flow above describes the expected ergonomics.

:::

---

## veBTC Boost Gauges and Matching Market

Although not specific to validators, veBTC boost gauges and the matching market directly affect how efficiently validators and LPs can attract votes.

- Each veBTC NFT has its own boost gauge (non-staking).
- veMEZO holders cast votes on these gauges using their veMEZO weight.

**Market Dynamics:**

- **BTC-heavy / MEZO-light participants:** Offer incentives (bribes) on their veBTC boost gauges to attract veMEZO votes to increase their multiplier (up to 5x).
- **MEZO-heavy / BTC-light participants:** Direct veMEZO votes to veBTC boost gauges to earn those incentives without holding BTC.

**For Validators:**

Operators with veBTC positions can enhance their effective voting power by pairing with veMEZO, whether self-owned or attracted via incentives. This can be used to:

- Increase their share of validator emissions by voting on their own gauge.

---

## LP Staking and Its Relationship to Validators

### What Is LP Staking?

LPs provide liquidity to DEX pools on Mezo and receive LP tokens representing their share of the pool. They can choose between two modes:

**1. Unstaked LP:**

- LPs keep LP tokens in their wallet.
- They earn their pro-rata share of swap fees directly from the pool.
- No MEZO emissions are routed to them.

**2. Staked LP:**

- LPs stake LP tokens into a staking gauge.
- Underlying fee share is diverted from the staked LP tokens to veBTC voters who vote for that gauge.
- LPs instead receive MEZO emissions routed to that gauge.

**The Economic Exchange:**

- LPs trade direct real yield (fees) for MEZO emissions.
- veBTC + veMEZO voters exchange their voting power for rights to those diverted fees and any incentives added to the gauge.

### How LP Staking Affects Validators

Through the splitter tree:

- The **chain splitter** determines how much of the reward pot goes to validator vs. non-validator activities. This ratio is currently fixed at 20% validators / 80% ecosystem and locked via governance.
- The **ecosystem splitter** determines how much of the non-validator pot goes to staking gauges (LP pools + MUSD savings) vs. ecosystem gauges. This ratio is currently fixed at 90% staking / 10% ecosystem and locked via governance.
- LP gauges then compete with the MUSD savings gauge, other LP gauges, and ecosystem gauges for emissions within their branch.

---

## Step-by-Step: LP Staking

1. **Provide liquidity:** Choose a pool on the Mezo DEX, deposit assets, and receive LP tokens.

2. **Locate the staking gauge:** Open the Earn or DEX frontend, find the staking gauge for the pool, and confirm it accepts LP tokens.

3. **Stake LP tokens:** Approve the contract and stake the desired amount. This diverts pool fees to the gauge's fee pot and distributes MEZO emissions to LP token stakers.

4. **veBTC voting on the LP gauge:** veBTC (+veMEZO) holders may vote on the LP gauge. Higher vote weight routes more MEZO emissions to the gauge and routes more pool fees/bribes to the voters.

5. **Claiming rewards:**
   - **LPs:** Claim MEZO emissions (allocated per epoch).
   - **veBTC voters:** Claim share of diverted swap fees and incentives.

---

## MUSD Savings Gauge

The MUSD savings rate is implemented as another staking gauge.

- Users deposit MUSD into a savings vault and receive a receipt token.
- If they stake the receipt token into the MUSD savings gauge:
  - Their direct MUSD yield is diverted into the gauge's fee pot.
  - They instead earn MEZO emissions routed to that gauge.
- veBTC holders who vote on the MUSD savings gauge receive:
  - MUSD yield (from interest, origination, and refinancing fees).
  - Any MEZO incentives routed to that gauge.

This gauge competes directly with LP gauges for staking-branch emissions and voting attention.

---

## Contact and Discord Support

For validator-specific questions, onboarding, and coordination: join the [Mezo Discord](https://discord.mezo.org) and join us in the #developers channel.
