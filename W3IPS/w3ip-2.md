---
w3ip: 2
title: Increase contract code size limit to 512K
author: Qi Zhou https://github.com/qizhou
discussions-to: https://github.com/web3q/W3IPs/pull/3
status: Draft
type: Standards
Track category: Core
created: 2022-02-17
requires: EIP-170
---


## Simple Summary

Increase contract code size limit to 512K so that we could use contract code to store binary large object (BLOB) close to 512K.

## Abstract

## Motivation

The motivation is to increase the code size limit to 512K and address some DDoS attack vectors in multiple contract code-related opcodes including

- CODESIZE
- CODECOPY
- EXTCODESIZE
- EXTCODECOPY
- EXTCODEHASH
- DELEGATECALL
- CALL
- CALLCODE
- STATICCALL
- CREATE
- CREATE2

## Specification

### Parameters

| Constant                  | Value            |
| ------------------------- | ---------------- |
| `MAX_CODE_SIZE_SOFT`      | 24K              |
| `MAX_CODE_SIZE_HARD`      | 512K             |
| `CHUNK_SIZE`              | 24K              |
| `CODE_STATKING_PER_CHUNK` | 1 W3Q = 1E18 WEI |
| `READ_GAS_PER_CHUNK`      | 700              |

For CODESIZE/CODECOPY/EXTCODESIZE/EXTCODEHASH, the gas is unchanged.

For EXTCODECOPY/CALL/CALLCODE/DELEGATECALL/STATICCALL, if the contract code size > `MAX_CODE_SIZE_SOFT`, then the opcode will take extra gas as

```
(CODE_SIZE - 1) // CHUNK_SIZE * READ_GAS_PER_CHUNK
```

where `//` is the integer divide operator.

For CREATE/CREATE2, if the newly created contract size > `MAX_CODE_SIZE_SOFT`, the opcode will require the following value of the token is in the contract after running the creation code (serve as staking)

`(CODE_SIZE - 1) // CHUNK_SIZE * CODE_STAKING_PER_CHUNK`.

## Rationale

The goal is to measure the CPU/IO cost of the contract read/write operations in terms of gas correctly so that resources will not be abused.

For code-size-related opcodes (CODESIZE/EXTCODESIZE), we would expect the client to implement a mapping from the hash of code to the size, so reading the code size of a large contract should not take extra resources.

For CODECOPY, the data is already loaded in memory (as part of CALL/CALLCODE/DELEGATECALL/STATICCALL), so we do not charge extra gas.

For EXTCODEHASH, the value is already in the account, so we do not charge extra gas.

For EXTCODECOPY/CALL/CALLCODE/DELEGATECALL/STATICCALL, since it will read extra data, we will additionally charge READ_GAS_PER_CHUNK per extra CHUNK_SIZE. Note that READ_GAS_PER_CHUNK = CALLGAS = EXTCODECOPYBASE = 700.

For CREATE/CREATE2, since storing a large contract will use the storage resources perpetually, we will require extra staking for creating such a contract. The staked value is part of the contract value and will be refunded once the contract is self-destructed. Note that a transfer from the contract to another account resulting in less than staking value will be failed.

By requiring staking, we could evaluate the maximum storage required for a node if extra contract space is used. Suppose the total supply of tokens is 1E8, the maximum storage size given current parameters will be 2.4TB.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
