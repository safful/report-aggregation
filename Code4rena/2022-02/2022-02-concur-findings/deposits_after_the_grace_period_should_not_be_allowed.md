## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Deposits after the grace period should not be allowed](https://github.com/code-423n4/2022-02-concur-findings/issues/251) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/Shelter.sol#L34
https://github.com/code-423n4/2022-02-concur/blob/main/contracts/Shelter.sol#L54


# Vulnerability details

## Impact
Function donate in Shelter shouldn't allow new deposits after the grace period ends, when the claim period begins. 
Otherwise, it will be possible to increase savedTokens[_token], and thus new user claim amounts will increase after some users might already have withdrawn their shares.

## Recommended Mitigation Steps
Based on my understanding, it should contain this check:
```solidity
  require(activated[_token] + GRACE_PERIOD > block.timestamp, "too late");
```

