## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [delete ps.stakeBalance](https://github.com/code-423n4/2021-07-sherlock-findings/issues/20) 

# Handle

gpersoon


# Vulnerability details

## Impact
In the function tokenUnload, ps.stakeBalance is only deleted if balance >0
e.g it is deleted if ps.stakeBalance > ps.firstMoneyOut
So if ps.stakeBalance ==  ps.firstMoneyOut then ps.stakeBalance will not be deleted.
And then a call to tokenRemove will revert, because it checks for ps.stakeBalance to be 0

## Proof of Concept
// https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/Gov.sol#L271
 function tokenUnload( IERC20 _token, IRemove _native, address _remaining ) external override onlyGovMain {
...
    uint256 balance = ps.stakeBalance.sub(ps.firstMoneyOut);
    if (balance > 0) {
      _token.safeTransfer(_remaining, balance);
      delete ps.stakeBalance;
    }
..
  delete ps.firstMoneyOut;

 function tokenRemove(IERC20 _token) external override onlyGovMain {
  ...
    require(ps.stakeBalance == 0, 'BALANCE_SET');


## Tools Used

## Recommended Mitigation Steps
Check what to do in this edge case and add the appropriate code.

