## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [staticcall may return true for an invalid _yieldSource](https://github.com/code-423n4/2021-10-pooltogether-findings/issues/45) 

# Handle

pauliax


# Vulnerability details

## Impact
Low-level calls like staticcall return true even if the account called is non-existent (per EVM design) so this hack in YieldSourcePrizePool constructor will not work in certain cases:
        // A hack to determine whether it's an actual yield source
        (bool succeeded, ) = address(_yieldSource).staticcall(
            abi.encodePacked(_yieldSource.depositToken.selector)
        );

You can try to pass an EOA address and see that it will return true.

## Recommended Mitigation Steps
Account existence must be checked prior to calling. A similar issue was submitted in a previous contest and assigned a severity of low, you can find more details here: https://github.com/code-423n4/2021-04-basedloans-findings/issues/16

