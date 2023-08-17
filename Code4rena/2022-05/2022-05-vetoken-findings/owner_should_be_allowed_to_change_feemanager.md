## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Owner should be allowed to change feeManager](https://github.com/code-423n4/2022-05-vetoken-findings/issues/99) 

# Lines of code

https://github.com/code-423n4/2022-05-vetoken/blob/main/contracts/Booster.sol#L129


# Vulnerability details

## Impact
Once Fee Manager has been set initially by owner, then owner has no power to change it. Owner should be allowed to change fees manager in case if he feels current fee manager is behaving maliciously

## Proof of Concept
1. Observe the setFeeManager function and see that only feeManager is allowed to change it once set initially

```
function setFeeManager(address _feeM) external {
        require(msg.sender == feeManager, "!auth");
        feeManager = _feeM;
        emit FeeManagerUpdated(_feeM);
    }
```

## Recommended Mitigation Steps
Change the setFeeManager function like below. Same can be done with other important functionality involving setArbitrator and setVoteDelegate

```
require(msg.sender == owner, "!auth");
```

