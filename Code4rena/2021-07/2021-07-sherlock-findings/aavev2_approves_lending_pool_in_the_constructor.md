## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [AaveV2 approves lending pool in the constructor](https://github.com/code-423n4/2021-07-sherlock-findings/issues/65) 

# Handle

pauliax


# Vulnerability details

## Impact
contract AaveV2 does not cache the lending pool, it retrieves it when necessary by calling a function getLp(). This is great as the implementation may change, however, this contract also approves an unlimited amount of want in the constructor:
   ILendingPool lp = getLp();
   want.approve(address(lp), uint256(-1));
so if the implementation changes, the approval will reset. This will break the deposit function as it will try to deposit to this new lending pool with 0 approval. 

For reference, function setLendingPoolImpl: https://github.com/aave/aave-protocol/blob/4b4545fb583fd4f400507b10f3c3114f45b8a037/contracts/configuration/LendingPoolAddressesProvider.sol#L58-L65 

Not sure how likely is that lending pool implementation will change so marking this as 'Low'.

## Recommended Mitigation Steps
Before calling lp.deposit check that the approval is sufficient and increase otherwise.

