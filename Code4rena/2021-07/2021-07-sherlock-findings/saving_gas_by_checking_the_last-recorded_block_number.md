## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Saving gas by checking the last-recorded block number](https://github.com/code-423n4/2021-07-sherlock-findings/issues/150) 

# Handle

shw


# Vulnerability details

## Impact

The `_accrueSherX` function of `LibSherX` and the `payOffDebtAll` function of `LibPool` can be called multiple times in the same block (from different users and transactions). If the current block number is the same as the last-recorded one, it is possible to save gas by early returning at the beginning of the functions.

## Proof of Concept

Referenced code:
[LibSherX.sol#L123-L141](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/libraries/LibSherX.sol#L123-L141)
[LibPool.sol#L84-L95](https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/libraries/LibPool.sol#L84-L95)

## Recommended Mitigation Steps

For example, consider re-writing `_accrueSherX` as follows:

```solidity
function _accrueSherX(IERC20 _token, uint256 sherXPerBlock) private returns (uint256 sherX) {
  PoolStorage.Base storage ps = PoolStorage.ps(_token);
  if (block.number == ps.sherXLastAccrued) {
    return 0;
  }
  sherX = block.number.sub(ps.sherXLastAccrued).mul(sherXPerBlock).mul(ps.sherXWeight).div(
    uint16(-1)
  );
  // need to settle before return, as updating the sherxperlblock/weight
  // after it was 0 will result in a too big amount (accured will be < block.number)
  ps.sherXLastAccrued = uint40(block.number);
  if (address(_token) == address(this)) {
    ps.stakeBalance = ps.stakeBalance.add(sherX);
  } else {
    ps.unallocatedSherX = ps.unallocatedSherX.add(sherX);
    ps.sWeight = ps.sWeight.add(sherX);
  }
}
```

