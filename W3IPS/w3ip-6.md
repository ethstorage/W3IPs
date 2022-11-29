---
w3ip: 6
title: Prune old block body
author: Qi Zhou https://github.com/qizhou
discussions-to: 
status: Draft
type: Standards
Track category: Core
created: 2022-11-20
requires: EIP-4444
---


## Simple Summary

Prune history block body, so that node disk requirement doesn't increase infinitely.


## Motivation

`EthStorage` passes key/value through calldata, so block body may grow infinitely, we solve it by pruning history block body.


## Specification

### Parameters

| Constant                  | Meaning            |
| ------------------------- | ---------------- |
| `*TendermintConfig.PruneConfig`              | If non nil, pruning will be enabled              |
| `DurationBlocks` | Assume pruning is enabled: 1. `DurationBlocks` is non zero, block body will be pruned after `DurationBlocks` blocks; 2.`DurationBlocks` is zero, block body will be pruned after `FullImmutabilityThreshold` blocks |


## Rationale

This is a simplified implementation of EIP-4444, in that we only prune block body which contains calldata, but keep headers and receipts(we may restrict the max size of receipt in the future). By doing this, the node disk requirement can be kept reasonably small while still convenient for dapps to query receipts. 

## Implementation

In the p2p layer, we still support legacy receipt format since it will make testing against existing network easier. In the storage layer, we drop support for legacy receipt format since we decided to do regenesis for this upgrade.

In order to make receipts complete even without block bodies, we store several redundant fields previously dependant on block bodies: `Type`/`TxHash`/`ContractAddress`, both for p2p(`receiptRLPComplete`) and storage(`storedReceiptRLPComplete`).

The other big change happens to the snapsync process, where we add a `queue.deliverWithPivot` so that block bodies before pivot are now optional instead of required. The safety holds since in `InsertHeaderChain` the last header will always be validated no matter whether `checkFreq` is `100` or `1`.

Currently there's no tool for importing and exporting of historical data yet, we may add it when necessary.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
