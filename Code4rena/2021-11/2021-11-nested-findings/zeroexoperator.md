## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [ZeroExOperator](https://github.com/code-423n4/2021-11-nested-findings/issues/3) 

# Handle

TomFrench


# Vulnerability details

## Impact

Reduced ease of verifying correctness

## Proof of Concept

`ZeroExOperator` uses the `Create2` library to deploy `ZeroExOperatorStorage`. `Create2` also exposes a `computeAddress` function which can be used to recalculate the address of `ZeroExOperatorStorage` but `ZeroExOperator` instead uses a homebrew calculation in `storageAddress`.

https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/operators/ZeroEx/ZeroExOperator.sol#L55-L65

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/57630d2a6466dff65aa7ca67b3fa23d5e6d1474a/contracts/utils/Create2.sol#L57-L64

The implementation is identical but using standard library code avoids the need for verification and minimises possible mistakes.

## Recommended Mitigation Steps

Replace `storageAddress` with

```
    function storageAddress(address own) public pure returns (address) {
       return Create2.computeAddress(
            bytes32("nested.zeroex.operator"),
            keccak256(type(ZeroExStorage).creationCode)
            own,
        );
    }

```

