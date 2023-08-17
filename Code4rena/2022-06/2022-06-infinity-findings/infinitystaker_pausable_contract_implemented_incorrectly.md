## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [InfinityStaker Pausable contract implemented incorrectly](https://github.com/code-423n4/2022-06-infinity-findings/issues/122) 

# Lines of code

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/staking/InfinityStaker.sol#L67
https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/staking/InfinityStaker.sol#L86-L90
https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/staking/InfinityStaker.sol#L116



# Vulnerability details

## Impact
`InfinityStaker.sol` implemented Pausable contract, but there's no functionality added to `pause` and `unpause` the contract.
If any attacker finds a way to exploit the contract and it's funds, at that time it will not let you pause the contract and funds can be lost.

## Proof of Concept
`InfinityStaker.sol` inhereted `Pausable.sol` of `Openzeppelin` and used `whenNotPaused` modifier for `stake()`, `unstake()` and `changeDuration()`, 

     function _pause() internal virtual whenNotPaused {
        _paused = true;
        emit Paused(_msgSender());
     }

 `_pause()` and `_unpause()` function of `Pausable.sol` used to `pause` and `unpause` the contract respectively and both has `internal` visibility, to use these functions it needs to access from the `infinityStaker.sol` internally.

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
add `pause` and `unpause` the contract function to `InfinityStaker.sol`

