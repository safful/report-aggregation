## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed
- valid

# [Project.checkPrecision() is passing 0 cost.](https://github.com/code-423n4/2022-08-rigor-findings/issues/253) 

# Lines of code

https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L903
https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L253
https://github.com/code-423n4/2022-08-rigor/blob/5ab7ea84a1516cb726421ef690af5bc41029f88f/contracts/Project.sol#L417


# Vulnerability details

## Impact
The task of zero cost is useless and there would be no subcontractors to accept such task as no payment after finish.

So if such task is added by mistake, it would require more time and effort to finish because builder must complete it by himself to recover tokens.


## Proof of Concept
It will work properly when _amount = 0.

```
function checkPrecision(uint256 _amount) internal pure { 
    // Divide and multiply amount with 1000 should be equal to amount.
    // This ensures the amount is not too precise.
    require(
        ((_amount / 1000) * 1000) == _amount,
        "Project::Precision>=1000"
    );
}
```

## Tools Used
Solidity Visual Developer of VSCode


## Recommended Mitigation Steps
Recommend changing like below.


```
function checkPrecision(uint256 _amount) internal pure { 
    require(_amount > 0, "zero amount");

    // Divide and multiply amount with 1000 should be equal to amount.
    // This ensures the amount is not too precise.
    require(
        ((_amount / 1000) * 1000) == _amount,
        "Project::Precision>=1000"
    );
}
```