## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Improper Upper Bound Definition on the Fee](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/255) 

# Handle

Jujic


# Vulnerability details

## Impact
The `rJoePerSec` does not have any upper or lower bounds. Values that are too large will lead to reversions in several critical functions.

## Proof of Concept
https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/RocketJoeStaking.sol#L151
```
function updateEmissionRate(uint256 _rJoePerSec) external onlyOwner {
        updatePool();
        rJoePerSec = _rJoePerSec;
        emit UpdateEmissionRate(msg.sender, _rJoePerSec);
    }
```

## Tools Used
Remix
## Recommended Mitigation Steps
Consider define  upper and lower bounds on the `_rJoePerSec`.

