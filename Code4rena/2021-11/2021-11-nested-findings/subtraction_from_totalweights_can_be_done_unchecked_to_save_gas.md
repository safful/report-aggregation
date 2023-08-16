## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Subtraction from totalWeights can be done unchecked to save gas](https://github.com/code-423n4/2021-11-nested-findings/issues/46) 

# Handle

GiveMeTestEther


# Vulnerability details


## Impact
FeeSplitter.so: totalWeights is the sum of shareholder weights and royaltiesWeight, therefore a subtraction of a shareholder weight or royaltiesWeight can be done unchecked because we can't underflow and save gas.


## Proof of Concept
https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/FeeSplitter.sol#L143

https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/FeeSplitter.sol#L169

## Tools Used
Manual Analysis

## Recommended Mitigation Steps

https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/FeeSplitter.sol#L143
can be rewritten as:

    function sendFees(IERC20 _token, uint256 _amount) external nonReentrant {
        _sendFees(_token, _amount, unchecked {totalWeights - royaltiesWeight});
    }

https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/FeeSplitter.sol#L169
can be written as
unchecked { _totalWeights -= shareholders[_accountIndex].weight; }

