---
w3ip: 4
title: Remove quadratic memory gas cost
author: Qi Zhou (@qizhou)
discussions-to: TBD
status: Draft
type: Standards Track
category: Core
created: 2022-04-09
---

## Simple Summary

Removal of quadratic memory gas cost that allows EVM to handle megabytes bytes in memory, enabling applications such as reading large files, large metadata composition, etc.

## Abstract
The current EVM memory gas cost equation per **call stack** is
```
gas_cost = mem_size_in_words * 3 + mem_size_in_words ^ 2 / 512,
```
which consists of a linear term and a quadratic term.

By removing the quadratic term, we propose the following the memory gas cost equation per **call stack**
```
gas_cost = mem_size_in_words * 3.
```


## Rationale
The second quadratic term can be negligible when memory size is small.  E.g., when `mem_size < 724`, the quadratic term is zero.  However, the second quadratic term heavily penalizes the usage of large memory.  The following lists some memory sizes and the costs of the quadratic terms:

Size (KB) | Linear Gas Term | Quadratic Gas Term | Total | Relative
-- | -- | -- | -- | --
1 | 96 | 2 | 98 | 0.02083333333
4 | 384 | 32 | 416 | 0.08333333333
16 | 1,536 | 512 | 2,048 | 0.3333333333
32 | 3,072 | 2,048 | 5,120 | 0.6666666667
64 | 6,144 | 8,192 | 14,336 | 1.333333333
99 | 9,504 | 19,602 | 29,106 | 2.0625
128 | 12,288 | 32,768 | 45,056 | 2.666666667
256 | 24,576 | 131,072 | 155,648 | 5.333333333
512 | 49,152 | 524,288 | 573,440 | 10.66666667
1,024 | 98,304 | 2,097,152 | 2,195,456 | 21.33333333
3,849 | 369,504 | 29,629,602 | 29,999,106 | 80.1875

while when `mem_size = 512K`, the quadratic term is about 10x higher than the linear term.  With the quadratic, the maximum allocated memory for an EVM call stack with 30M gas limit is about 3849KB.

The memory gas metering of using a quadratic term to prevent EVM from using large memory is unreasonable due to the following reasons:
1. The cost of allocating memory for a decent size is linear.  As long as the memory can be stored in physical memory (i.e., no memory swapping), the cost is linear in terms of size.  Taking 50M gas as block limit as an example, without quadratic term, the maximum memory size is 30M / 3 * 32 = 305MB, which be easily fit into the memory in a commodity PC.
2. The cost of expanding memory generally relies on the cost of append() operation, whose amortized cost is O(1).  Even the worse case is O(N), we can show that the worst case attack will result in linear gas term.
3. If an attacker wants to perform a memory allocation attack, the attacker can employ a strategy to circumvent the quadratic term by creating multiple call stacks.  The basic gas cost of calling a contract is 700, and EVM allows 1024 call depth.  This means that an attacker can use TX gas limit / 1024 - 700 (e.g., 30M / 1024 - 700 = 28596) to allocate the memory, which can allocate up to 1024 * 99KB ~ 99MB.

### Modeling of Memory Allocation / Expansion
Consider the following simplified memory model, where
- the copy minimal size is 32 bytes with 3 gas per copy;
- when expanding the memory to size, we will create a new memory with a capacity to the nearest 32 * 2 ** N >= size, where N is an integer, and then copy the original memory to the new memory.

As a result, given a memory size, we could find N, so that the worst copy cost from memory size zero to current memory size will be 3 * 2 ** N, which is close to existing linear term.

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
