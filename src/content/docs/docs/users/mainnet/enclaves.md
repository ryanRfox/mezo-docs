---
title: Enclaves
description: How institutional BTC deposits work on Mezo through enclaves
topic: users
---

Mezo Enclaves are an institutional deposit pathway that allows institutions with regulatory custody requirements to deploy BTC on Mezo without using the standard bridge.

## How Enclaves Work

Institutions operating under custody regulations may not be able to deposit BTC through Mezo's standard bridge. Enclaves solve this by allowing BTC to be represented on Mezo while the underlying Bitcoin remains in custody arrangements that satisfy regulatory requirements.

Key points:

- Each enclave is individually allowlisted by the Mezo governance multisig
- Enclave deposits are backed by associated legal agreements that define bridge amounts and usage limitations
- All enclave deposits are still backed by real Bitcoin UTXOs on the Bitcoin chain

## Verifying Enclave BTC Backing

Every enclave deposit is backed by UTXOs on the Bitcoin blockchain. These UTXOs are captured in the triparty bridge deposit transaction and maintained in on-chain state, meaning the underlying BTC backing is transparent and verifiable.

Developers can query enclave UTXO data using the `getTripartyUtxos` function to verify the Bitcoin backing any enclave position. See the [developer bridge documentation](/docs/developers/bridge/mezo-bridge) for technical details.
