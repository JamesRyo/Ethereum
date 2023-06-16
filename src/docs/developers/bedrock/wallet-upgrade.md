---
title: Wallet upgrade guide
lang: en-US
---

The Bedrock release introduces a few changes that affect wallets.

- Some of the JSON RPC methods have changed.
- A transaction can now be in one of three statuses.
- Transaction fees have to acount for EIP 1559 support. 


## JSON RPC

These methods have been removed:

- `eth_getBlockRange`: Use `eth_getBlockByNumber` in a batch request instead.
- `rollup_getInfo`: None of the information returned by this method exists on Bedrock, so there is no replacement for this method.
- `rollup_gasPrices`: Use [`eth_gasPrice`](https://docs.alchemy.com/reference/eth-gasprice) instead for the L2 gas price. 
  For the L1 gas price, you can call the [`GasPriceOracle`'s `l1BaseFee` function](https://optimistic.etherscan.io/address/0x420000000000000000000000000000000000000F#readContract#F5).
  If you want to estimate the cost of a transaction, you can [use the SDK](https://github.com/ethereum-optimism/optimism-tutorial/tree/main/sdk-estimate-gas).


## Transaction status

We use the same vocabulary as the Beacon Chain to describe block finality. 
Blocks (and the transactions within them) can be in one of the following states:

- `unsafe`, meaning that the block has been received via gossip but has not yet been submitted to L1. Unsafe blocks can be reorged if L1 reorgs, or the sequencer reorgs.
- `safe`, meaning that the block has been submitted to L1. Safe blocks can also be reorged if L1 reorgs.
- `finalized`, meaning that the block has reached sufficient depth to be considered final. Finalized blocks cannot be reorged.

To get the status of a specific block, use [`eth_getBlockByNumber`](https://docs.alchemy.com/reference/eth-getblockbynumber) with the "block number" `finalized`.
If the last finalized block is the same or greater than the block with the transaction whose status you need, then it is finalized.
If not, use [`eth_getBlockByNumber`](https://docs.alchemy.com/reference/eth-getblockbynumber) with the "block number" `safe`. If that block is the same or after the one with the transaction, then it is `safe` (highly unlikely to be reorganized, but it could happen). Otherwise, it is `unsafe`.


## Transaction fees

In Bedrock we support [EIP 1559](https://eips.ethereum.org/EIPS/eip-1559).
Therefore, the L2 execution fee is composed of two components: a fixed (per-block) base fee and a user selected priority fee.

To enable your users to select a priority fee, you can [build a priority fee estimator]
(https://docs.alchemy.com/docs/how-to-build-a-gas-fee-estimator-using-eip-1559).
If you already have estimating code you use for L1 Ethereum, you can just use that.

Note that on OP Mainnet the priority fee tends to be very low. 
As I am writing this, a priority fee of 0.0002 gwei is sufficient ([see here](https://optimism.io/gas-tracker) to get the current values).

To display the entire estimated cost of a transaction we recommend you [use the SDK](https://github.com/ethereum-optimism/optimism-tutorial/tree/main/sdk-estimate-gas). 

