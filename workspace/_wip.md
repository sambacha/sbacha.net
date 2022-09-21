---
title: "Evm Lowlevel"
date: 2021-10-15T17:27:03-07:00
draft: true
---

## Functions

If you call a Solidity function normally, it returns ABI encoded data, which is always packaged up into 32 byte slots. So the valid return of any Solidity function with be congruent to 0 mod 32. When an error occurs, it prepends the normal ABI encoded data (0 mod 32) with a 4 byte selector (currently only sighash("Error(string)") is supported but the EIP allows for arbitrary error types in the future, so it's best to check the first 4 bytes match Error(string)), so that makes the number of bytes returned congruent to 4 mod 32.
