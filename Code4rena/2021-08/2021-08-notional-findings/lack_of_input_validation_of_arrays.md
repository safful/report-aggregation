## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [lack of input validation of arrays](https://github.com/code-423n4/2021-08-notional-findings/issues/43) 

# Handle

JMukesh


# Vulnerability details

## Impact
function migrateBorrowFromCompound(
        address cTokenBorrow,
        uint256 cTokenRepayAmount,
        uint16[] memory notionalV2CollateralIds,
        uint256[] memory notionalV2CollateralAmounts,
        BalanceActionWithTrades[] calldata borrowAction
    ) ;

if the  array length of notionalV2CollateralId ,  notionalV2CollateralAmounts and  borrowAction is not equal it can lead to an error

## Proof of Concept

https://github.com/code-423n4/2021-08-notional/blob/4b51b0de2b448e4d36809781c097c7bc373312e9/contracts/external/adapters/CompoundToNotionalV2.sol#L24


## Tools Used

manual review

## Recommended Mitigation Steps
check the input array length

