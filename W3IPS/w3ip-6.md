---
w3ip: 6
title: Prune historical block bodies
author: Qi Zhou https://github.com/qizhou
discussions-to: 
status: Draft
type: Standards
Track category: Core
created: 2022-11-20
requires: EIP-4444
---


## Simple Summary

Prune historical block bodies to reduce the node disk requirement.


## Motivation

`EthStorage` uses calldata in the transactions (and thus in block bodies) to carry uploaded data.  This means that the block bodies contain all uploaded data, which can be 10TB or 100TB, or more if the use of EthStorage grows. The proposal solves the issue by pruning historical block bodies.


## Specification

### Parameters

| Constant                  | Meaning            |
| ------------------------- | ---------------- |
| `*TendermintConfig.PruneConfig`              | If non nil, pruning will be enabled              |
| `DurationBlocks` | Assume pruning is enabled: 1. `DurationBlocks` is non zero, block body will be pruned from block 0 to block lastest - `DurationBlocks` block; 2.`DurationBlocks` is zero, block body will be pruned from block 0 to block latest - `FullImmutabilityThreshold` |


## Rationale

This is a simplified implementation of EIP-4444, in that we only prune block bodies that contain potentially large calldata in EthStorage case, but we keep headers and receipts (the restriction for the max size of receipts may be imposed in the future). By doing this, the node disk requirement can be kept reasonably small (about 1TB) while the node is still convenient for dapps to query receipts. 

The headers are kept in the version so that light clients are supported easily.  For the disk requirement of the headers, suppose the block time is 5s with each block size 1KB, the disk requirement per year will be 365 * 24 * 3600 * / 1KB ~ 6GB.

## Implementation

In the p2p layer, the legacy receipt format is upgraded to be a self-contained complete version without deriving the information from block bodies.  However, the implementation still supports receiving legacy receipt format since it will make testing against the existing network easier. In the storage layer, we drop support for the legacy receipt format since we assume that after this upgrade, the nodes will start with a fresh database.

In order to make receipts complete even without block bodies, we store several redundant fields previously dependent on block bodies: `Type`/`TxHash`/`ContractAddress`, both for p2p (`receiptRLPComplete`) and storage (`storedReceiptRLPComplete`) layers.

The other major change is in the snapsync process, where we add a `queue.deliverWithPivot` so that block bodies before a pivot are now optional instead of required during synchronization. The safety holds since in `InsertHeaderChain` the last header will always be validated no matter whether `checkFreq` is `100` or `1`.

Only snapsync will be supported after the upgrade. Other synchronization methods such as fullsync will not be supported anymore.

Currently, there is no tool for importing and exporting historical data yet, we may add it when necessary.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).


