---
title: MUSD Stablecoin Overview
description: 'Everything you need to know about MUSD, Mezo’s Bitcoin-backed stablecoin.'
topic: users
---

## What is MUSD?

MUSD is a permissionless stablecoin 100% backed by Bitcoin reserves and designed to maintain a 1:1 peg with the U.S. dollar. It is the native stablecoin on Mezo, accessible via Mezo's 'Borrow' feature or decentralized exchanges on Mezo Network.

Anyone can mint MUSD by depositing BTC into Mezo borrow, thus creating a loan position. Bitcoin collateral for MUSD positions is publicly verifiable onchain, and proof-of-reserves are viewable 24-7. Users can close their MUSD positions by returning the borrowed MUSD and accumulated interest to receive their initial Bitcoin collateral.

![MUSD Lets You Keep and Spend BTC](/docs/images/musd/musd-keep-and-spend-btc.png)

:::note
The minimum loan size is **1,800 MUSD** of borrowed debt. With the 200 MUSD gas deposit, your total debt at opening is at least 2,000 MUSD. This is a minimum *borrow* amount, not a collateral floor — the collateral you need depends on your desired collateral ratio.
:::

### MUSD solutions

On Mezo, MUSD and Borrow provide solutions to several lending market challenges:

1. **Permissionless credit** — Access up to 90% of your BTC holdings. Keep your BTC, tap into your Bitcoin equity, and pay your loan back whenever.
2. **Onchain and verifiable** — Loans are created onchain and [publicly verifiable 24/7](https://explorer.test.mezo.org/address/0x118917a40FAF1CD7a13dB0Ef56C86De7973Ac503?tab=txs&ref=blog.mezo.org). Close your position whenever you want to receive your underlying Bitcoin collateral.
3. **Fixed borrow rates** — MUSD borrow rates are fixed for the life of the loan, starting at 1–5%. Lock in your low rate, and don't worry about the market.

This documentation provides details about the MUSD architecture, how it fits into the Mezo ecosystem, risks and mitigations, and guides through how to access MUSD. 

## How MUSD works

MUSD uses a CDP (collateralized debt position) model.

* Every outstanding MUSD is redeemable for Bitcoin.
* \$1 in BTC collateral can be used to mint 1 MUSD

The mint-and-redeem model helps maintain the \$1 peg in volatile environments. For example, MUSD may trade on the market at a premium or discount to its \$1 stable value. Below are the scenarios for how the peg can be re-established.

* If MUSD is trading at a discount of \$0.99, arbitragers can buy MUSD on the market and redeem it for \$1 in underlying BTC. Users with a loan position can do this for no additional cost. Those without a loan position must pay a 0.75% redemption fee, which remains profitable until MUSD reaches a price of \$0.995.
* If MUSD is trading at a premium of \$1.05, arbitragers can mint MUSD by supplying BTC to the protocol and sell the minted MUSD on the market for a profit; selling into another dollar-equivalent stablecoin like USDT or USDC. This scenario remains profitable until MUSD returns to a price of \$1.005.

![](/docs/images/musd/musd-redemption-and-peg-process.png)

To ensure the peg is maintained during market volatility, sufficient BTC collateral must always back the outstanding MUSD. Outstanding loan positions must maintain a collateral ratio of above 110%, and the system has built-in liquidation mechanisms and stability pools to enforce this. These risk mitigations ensure that even with high LTVs, the system remains secure and resilient against market volatility.

Details on liquidations and risks can be read in further detail in [Liquidations & Redemptions](/docs/users/musd/liquidation-mechanics).

### Benefits of the MUSD model

**Supply-Sided Market**

MUSD is a single-sided lending market. The mint and redeem model that MUSD uses benefits the system as a whole by creating liquidity from the supply side rather than relying on pre-existing dollar pools. Instead of needing a fixed reservoir of dollars like traditional two-sided lending protocols that require a dynamic market driven interest rate to balance supply and demand for asset pairs, MUSD is minted directly from Bitcoin collateral. This approach means that as more users deposit Bitcoin to mint MUSD, liquidity grows organically in line with demand. 

The system isn’t constrained by traditional dollar liquidity—it self-generates its supply, ensuring that liquidity is always available for new depositors of BTC collateral.

**Low Borrowing Rates**

Because MUSD is created through the minting of “new money” backed by Bitcoin collateral, borrowers benefit from fixed interest rates. The process sidesteps the need for competitive borrowing from pre-existing dollar pools, which often drives up costs due to market pressures. Instead, the fixed, low rates (as low as 1%) stem from the efficient, collateralized minting mechanism of MUSD. 

With MUSD, users can unlock the value of their Bitcoin at a lower cost, making MUSD an attractive alternative to conventional lending markets where rates can be highly variable and significantly higher.

**Extremely High LTV**

One of the standout features of MUSD is its ability to support extremely high loan-to-value (LTV) ratios. The collateralization requirement is 110% in its normal mode for total collateralization ratio of all loans or 150% in other modes. Borrowers can potentially access up to 90% of the value of their Bitcoin holdings, but it is recommended to always calculate the risk of your loan especially for lower collateralization ratios. This high LTV is crucial for Bitcoin holders because it allows them to maximize the liquidity they can unlock from their assets without having to sell them. 

## MUSD comparison to existing stablecoins

The stablecoin market is broad, ranging from fiat-backed stables to synthetic stables to other algorithmic CDPs. MUSD aims to address the risks inherent in each category with its pure Bitcoin backing.

![](/docs/images/musd/built-different.avif)

| Type | Examples | Key Risk | MUSD Difference |
| --- | --- | --- | --- |
| **Fiat-backed** | USDT, USDC | Centralized custody, blacklisting, opaque reserves | 100% BTC-backed, onchain proof of reserves |
| **Synthetic** | USDe | Relies on centralized exchanges and basis trades; untested under stress | No exchange dependency; collateral is native BTC |
| **Multi-asset CDP** | USDS (Sky) | Basket of volatile altcoins can destabilize the peg | Single-collateral (BTC) — simpler, less correlated risk |
