## Tags

- bug
- 1 (Low Risk)
- mStableYieldSource
- sponsor confirmed

# [No input validation for  while setting up value for immutable state variables](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/15) 

# Handle

JMukesh


# Vulnerability details

## Impact
Since  immutable state variable cant be change after initialization in constructor, their value should be checked before initialization


    constructor(ISavingsContractV2 _savings) ReentrancyGuard() {

           // @audit --> there should be a input validation

        // As immutable storage variables can not be accessed in the constructor,
        // create in-memory variables that can be used instead.
        IERC20 mAssetMemory = IERC20(_savings.underlying());

        // infinite approve Savings Contract to transfer mAssets from this contract
        mAssetMemory.safeApprove(address(_savings), type(uint256).max);

        // save to immutable storage
        savings = _savings;
        mAsset = mAssetMemory;

        emit Initialized(_savings);
    }


## Proof of Concept

https://github.com/pooltogether/pooltogether-mstable/blob/0bcbd363936fadf5830e9c48392415695896ddb5/contracts/yield-source/MStableYieldSource.sol#L45

## Tools Used
no tool used

## Recommended Mitigation Steps
add a require condition to validate input values

