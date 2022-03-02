---
w3ip: 3
title: Directory Standard
author: Qi Zhou (@qizhou)
type: Standards Track
category: W3IP
status: Draft
created: 2022-02-21
---

## Simple Summary

A standard interface for filesystem directories.

## Abstract

The following standard allows for the implementation of a standard API for filesystem directories within smart contracts.
This standard provides basic functionality to read/write binary objects for any size, as well as allow reading/writing chunks of the object if the object is too large to fit in a single transaction.

## Motivation

A standard interface allows any binary objects on EVM-based blockchain to be re-used by other dApps.

## Specification

## Directory

### Methods

#### write

Writes binary `data` to the file `name` in the directory by an account with write permission.

```
function write(bytes memory name, bytes memory data) external payable
```

#### read

Returns the binary `data` from the file `name` in the directory and existence of the file.

```
function read(bytes memory name) external view returns (bytes memory, bool)
```

#### size

Returns the size of the `data` from the file `name` in the directory and the number of chunks of the data.

```
function size(bytes memory name) external view returns (uint256, uint256)
```

#### remove

Removes the file `name` in the directory and returns the number of chunks removed (0 means the file does not exist) by an account with write permission.

```
function remove(bytes memory name) external returns (uint256)
```

#### countChunks

Returns the number of chunks of the file `name`.

```
function countChunks(bytes memory name) external view returns (uint256);
```

#### writeChunk

Writes a chunk of data to the file by an account with write permission. The write will fail if `chunkId > numOfChunks`, i.e., the write must append the file or replace the existing chunk.

```
 function writeChunk(bytes memory name, uint256 chunkId, bytes memory data) external payable;
```

#### readChunk

Returns the chunk data of the file `name` and the existence of the chunk.

```
function readChunk(bytes memory name, uint256 chunkId) external view returns (bytes memory, bool);
```

#### chunkSize

Returns the size of a chunk of the file `name` and the existence of the chunk.

```
function chunkSize(bytes memory name, uint256 chunkId) external view returns (uint256, bool);
```

#### removeChunk

Removes a chunk of the file `name` and returns `false` if such chunk does not exist. The method should be called by an account with write permission.

```
function removeChunk(bytes memory name, uint256 chunkId) external returns (bool);
```

## Implementation

An example of the implementation can be found [here](https://github.com/web3q/evm-large-storage/blob/main/contracts/examples/FlatDirectory.sol)

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).