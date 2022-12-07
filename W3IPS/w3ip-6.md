---
w3ip: 6
title: Add external calls within consensus mechanism
author: 0xhhh https://github.com/cyl19970726
discussions-to: 
status: Draft
type: Standards
Track category: Core
created: 2022-06-25
requires: 
---


## Simple Summary

Support Web3Q to directly call Ethereum's on-chain data in transactions.

## Abstract

Any account can make a cross-chain data call by calling the precompiled contract in the initiated transaction. If the Web3Q Proposer receives a transaction with cross-chain call data, it will put the result of the cross-chain call in `receipt.externalCallResult`, and Participate in the generation of the merkle root of receipt trie, so as to include the result of the cross-chain call into the proposer block. 

The validators of Web3Q will re-execute these transactions including the cross-chain call and participate in the generation of a new receipt root with their results.
Then the validator can verify the correctness of the cross-chain call by verifying whether the receipt root is consistent with the receipt root in the proposer block.
Therefore, Web3Q cross-chain call is combined with Web3Q's block generation consensus, and the security of cross-chain call is equivalent to that of Web3Q.

## Motivation

1. Web3Q is the sidechain of ethereum,  each Web3Q validator needs to run an Ethereum node, so the validators is born with the ability to verify the results of cross-chain calls.
2. The security of Web3Q cross-chain calls is equivalent to the security of web3q, which can greatly improve the cross-chain security of the Web3Q bridge.

## Specification

### Perform cross-chain calls within a contract
We implemented a precompiled contract at address `0x00000000000000000000000000000000033303` to perform cross-chain calls.
At present, we provide a `getLogByTxHash` method in this precompiled contract to call the log data of ethereum.
```
getLogByTxHash(uint256 chainId , bytes32 txHash, uint256 logIdx ,uint256 maxDataLen, uint256 confirms)
```
#### params
- chainId: The chainId of the target chain of the cross-chain call.
- txHash: Transaction hash on the target chain.
- logIdx: The index of the log to be obtained in the corresponding transaction.
- maxDataLen: Maximum length of log.data.
- confirms: The number of block confirmations required for this transaction
#### returns
##### no happen error
- Encode `log.address` into an `address` type through `abi.encode`
- Encode `log.topics` into an `bytes32[]` type through `abi.encode`
- Encode `log.data` into an `bytes` type through `abi.encode`
##### happen error
- Encode `error msg` into an `string` type through `abi.encode`
#### example: decode result
```
    bytes memory payload = abi.encodeWithSignature("getLogByTxHash(uint256,bytes32,uint256,uint256,uint256)", chainId,txHash,logIdx,maxDataLen,confirms);
    (bool succeed,bytes memory log1) = crossChainCallContract.call(payload);
    if (!succeed) {
        string memory errMsg = abi.decode(log1, (string));
        return;
    }
        
    (address c, bytes32[] memory _topics, bytes memory data) = abi.decode(log1, (address, bytes32[], bytes));
```

### view the result of the cross-chain via RPC
Web3Q added an `externalCallResult` property to the results of `eth_getTransaction` and `eth_getTransactionReceipt`.

### Parameters

| Constant                  | Value            |
| ------------------------- | ---------------- |
| `ExternalCallByteGasCost` | 3               |
| `ExternalCallGas`         | 200000  |

For the gas cost of cross-chain calls, follow the formula below：
```Gas Cost = ExternalCallGas + len(ExternalCallResult) * ExternalCallByteGasCost```.

## Rationale

Web3Q cross-chain call is combined with Web3Q's block generation consensus, and the security of cross-chain call is equivalent to that of Web3Q.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
