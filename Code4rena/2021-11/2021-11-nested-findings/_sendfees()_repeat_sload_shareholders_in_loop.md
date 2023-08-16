## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [_sendFees() Repeat SLOAD shareholders In Loop](https://github.com/code-423n4/2021-11-nested-findings/issues/57) 

# Handle

Meta0xNull


# Vulnerability details

## Impact
_sendFees() repeat Read Storage variable shareholders. Every Storage read is expensive. 

## Proof of Concept
https://github.com/code-423n4/2021-11-nested/blob/main/contracts/FeeSplitter.sol#L227-L232

## Tools Used
Manual Review

## Recommended Mitigation Steps
Read the values from storage once, cache them in local variables and then read them again using the local variables. For example:

Shareholder[] shareholders_temp = shareholders;
        for (uint256 i = 0; i < shareholders_temp.length; i++) {
            _addShares(
                shareholders_temp[i].account,
                _computeShareCount(_amount, shareholders_temp[i].weight, _totalWeights),
                address(_token)
            );

