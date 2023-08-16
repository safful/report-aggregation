## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- disagree with severity

# [Attacker can trigger pool sync leading to user fund loss](https://github.com/code-423n4/2021-07-spartan-findings/issues/120) 

# Handle

0xRajeev


# Vulnerability details

## Impact

An attacker can front-run any operation that depends on the pool contract's internal balance amounts being unsynced to pool's balance on token/base contracts effectively nullifying the transfer of base/tokens for those operations. This will make _getAddedBaseAmount() and _getAddedTokenAmount() return 0 (because the balances are synced) from such operations. 

Impact: The affected operations are: addForMember(), swapTo() and mintSynth() which will all take the user funds to respective contracts but will treat it as 0 (because of the syncing) and thus not add liquidity, return swapped tokens or mint any synths to the affected users. User loses deposited funds to the contract.

## Proof of Concept

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L308-L312

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L261-L270

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L272-L281

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L216-L220

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L231

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Pool.sol#L174-L175

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/Router.sol#L279

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add access control to sync() function so that only Router can call it via addDividend().

