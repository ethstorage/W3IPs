---
w3ip: 5
title: Allow fractional lock-for-storage amount
author: Qi Zhou https://github.com/qizhou
discussions-to: 
status: Draft
type: Standards
Track category: Core
created: 2022-04-14
requires: W3IP-2
---


## Simple Summary

Allow fractional amount to lock for storage, which is more friendly to users.

## Abstract

## Motivation

## Specification

### Parameters

| Constant                  | Value            |
| ------------------------- | ---------------- |
| `CHUNK_SIZE`              | 24K              |
| `CODE_STATKING_PER_CHUNK` | 1 W3Q = 1E18 WEI |

For CREATE/CREATE2, if the newly created contract size > `MAX_CODE_SIZE_SOFT`, the original opcode will require the following value of the token is in the contract after running the creation code (serve as staking)

`(CODE_SIZE - 1) // CHUNK_SIZE * CODE_STAKING_PER_CHUNK`.

The new amount required will be 0 if `CODE_SIZE <= CHUNK_SIZE`, otherwise
`(CODE_SIZE - CHUNK_SIZE) * CODE_STAKING_PER_CHUNK // CHUNK_SIZE`.

## Rationale

The purpose is to reduce the lock for storage amount to while maintaining our the maximum storage limit using lock for storage.  The original formula is essentially the same as
`ceiling((CODE_SIZE - CHUNK_SIZE) / CHUNK_SIZE) * CODE_STAKING_PER_CHUNK`.

The proposed formula will reduce the lock for storage by removing the `ceiling()` function as
`int((CODE_SIZE - CHUNK_SIZE) / CHUNK_SIZE * CODE_STAKING_PER_CHUNK) = (CODE_SIZE - CHUNK_SIZE) * CODE_STAKING_PER_CHUNK // CHUNK_SIZE`.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
