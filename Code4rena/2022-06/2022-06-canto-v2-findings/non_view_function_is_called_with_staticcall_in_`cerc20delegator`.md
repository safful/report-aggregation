## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Non view function is called with staticcall in `CErc20Delegator`](https://github.com/code-423n4/2022-06-canto-v2-findings/issues/112) 

# Lines of code

https://github.com/Plex-Engineer/lending-market-v2/blob/443a8c0fed3c5018e95f3881a31b81a555c42b2d/contracts/CErc20Delegator.sol#L237
https://github.com/Plex-Engineer/lending-market-v2/blob/443a8c0fed3c5018e95f3881a31b81a555c42b2d/contracts/CErc20Delegator.sol#L246


# Vulnerability details

## Impact

When using CToken implementation with CErc20Delegator, the functions `borrowRatePerBlock` and `supplyRatePerBlock` will revert when the underlying functions try to update some states.

## Detail

The v1 of [borrowRatePerBlock](https://github.com/Plex-Engineer/lending-market-v2/blob/443a8c0fed3c5018e95f3881a31b81a555c42b2d/contracts/CToken.sol#L208) and [supplyRatePerBlock](https://github.com/Plex-Engineer/lending-market-v2/blob/443a8c0fed3c5018e95f3881a31b81a555c42b2d/contracts/CToken.sol#L216) were view functions, but they are not anymore. The `CErc20Delegator` is still using `delegateToViewImplementation` for those functions. Those functions can be used, as long as the implementation does not update any state variables, i.e. [the block number increase since the last update is less or equal to the `updateFrequency`](https://github.com/Plex-Engineer/lending-market-v2/blob/443a8c0fed3c5018e95f3881a31b81a555c42b2d/contracts/NoteInterest.sol#L141). However, when these functions are called after sufficient blocks are mined, they are going to revert.
Although one can still call the implementation using [`delegateToImplementation`](https://github.com/Plex-Engineer/lending-market-v2/blob/443a8c0fed3c5018e95f3881a31b81a555c42b2d/contracts/CErc20Delegator.sol#L437), it is not a good usability, especially if those functions are used for external user interface.


## Proof Of Concept

[gist for the test](https://gist.github.com/zzzitron/37fb99cebed786b4c983d20a76e8793e#file-2022-06-newblockchain-v2-poc-ctoken-test-ts-L49-L62)

The gist shows a simple test. It calls `borrowRatePerBlock` and `supplyRatePerBlock` first time, it suceeds. Then, it mines for more than 300 times, which is the `updateFrequency` parameter. Then it calls again then fails.

Notes on the test file:
- The setup is taken from `tests/Treasury/Accountant.test.ts`
- using `solidity` from ethereum-waffle for chai to use `reverted`
  ```
  // in hardhat.config.js
  import chai from "chai";
  import { solidity } from "ethereum-waffle";

  chai.use(solidity);
  ```

## Tools Used

hardhat


## Recommended Mitigation Steps

Instead of using `delegateToViewImplementation` use `delegateToImplementation`. Alternatively, implement view functions to query these rates in `NoteInterest.sol` and `CToken.sol`. It will enable to query the rates without spending gas.

