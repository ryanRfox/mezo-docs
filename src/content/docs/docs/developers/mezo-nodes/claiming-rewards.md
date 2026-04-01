---
title: Claiming MEZO Rewards
description: Guide for validators on how to claim MEZO rewards and manage reward beneficiaries.
topic: developers
---

Validators earn MEZO rewards for securing the network. This guide explains how to claim these rewards and manage reward beneficiaries.

## Overview

Each validator has an associated gauge in the [ValidatorsVoter contract](https://explorer.mezo.org/address/0xe99a9ad5Ed26BD30e4DB25397f378817e9b9515a). Rewards are claimed through the ValidatorsVoter contract by specifying the validator's gauge address.

Each gauge has a single beneficiary address that is authorized to claim rewards. By default, the beneficiary is the validator operator address registered in the [Proof of Authority (PoA) contract](https://explorer.mezo.org/address/0x7B7C000000000000000000000000000000000011?tab=read_contract#ca1e7819).

:::note[Reward Release Schedule]
Allocated epoch rewards are not immediately available for claim—they are gradually released over the epoch. For example, if rewards are allocated to a gauge at the beginning of epoch N, 100% of epoch N rewards will be available at the epoch's end.
:::

## Claiming Rewards

To claim your MEZO rewards, follow these steps:

1. **Connect your operator address wallet**
   Use the wallet associated with your validator operator address.

2. **Determine your gauge address**
   Call `ValidatorsVoter.validatorToGauge()` with your operator address as the parameter.
   - Contract: [ValidatorsVoter](https://explorer.mezo.org/address/0xe99a9ad5Ed26BD30e4DB25397f378817e9b9515a)
   - Method: `validatorToGauge(address validator)`

3. **Claim rewards**
   Call `ValidatorsVoter.claimRewards()` with your gauge address as the parameter.
   - Contract: [ValidatorsVoter](https://explorer.mezo.org/address/0xe99a9ad5Ed26BD30e4DB25397f378817e9b9515a)
   - Method: `claimRewards(address gauge)`

These steps can be performed through the [Mezo block explorer](https://explorer.mezo.org) or via a custom script.

### Example via Block Explorer

1. Navigate to the [ValidatorsVoter contract](https://explorer.mezo.org/address/0xe99a9ad5Ed26BD30e4DB25397f378817e9b9515a)
2. Go to the "Read Contract" tab
3. Call `validatorToGauge` with your operator address to get your gauge address
4. Switch to the "Write Contract" tab
5. Connect your operator wallet
6. Call `claimRewards` with your gauge address

## Changing Reward Beneficiary

Validators can designate a different address to receive MEZO rewards. This is useful for separating operational and treasury functions.

To change the reward beneficiary:

1. **Connect your operator address wallet**
   Use the wallet associated with your validator operator address.

2. **Determine your gauge address**
   Call `ValidatorsVoter.validatorToGauge()` with your operator address as the parameter.

3. **Switch beneficiary**
   Call `Gauge.switchRewardsBeneficiary()` with the new beneficiary address as the parameter.
   - Contract: Your specific Gauge contract (obtained from step 2)
   - Method: `switchRewardsBeneficiary(address newBeneficiary)`

4. **Claim from the new beneficiary**
   After changing the beneficiary, all future claims must be executed using the new beneficiary wallet.

:::caution[Best Practice]
It's recommended to claim all available rewards with the current beneficiary before switching to a new one. If you switch beneficiaries with unclaimed rewards, those rewards will remain associated with the old beneficiary address and must be claimed separately.
:::

## Contract Addresses

- **ValidatorsVoter**: [`0xe99a9ad5Ed26BD30e4DB25397f378817e9b9515a`](https://explorer.mezo.org/address/0xe99a9ad5Ed26BD30e4DB25397f378817e9b9515a)
- **PoA Contract**: [`0x7B7C000000000000000000000000000000000011`](https://explorer.mezo.org/address/0x7B7C000000000000000000000000000000000011?tab=read_contract#ca1e7819)

## Additional Resources

- [Mezo Block Explorer](https://explorer.mezo.org)
- [Validator Information](./validators)
- [Validator Kit](./validator-kit)
