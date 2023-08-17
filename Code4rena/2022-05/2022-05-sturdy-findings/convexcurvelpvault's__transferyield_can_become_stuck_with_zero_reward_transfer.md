## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [ConvexCurveLPVault's _transferYield can become stuck with zero reward transfer](https://github.com/code-423n4/2022-05-sturdy-findings/issues/79) 

# Lines of code

https://github.com/code-423n4/2022-05-sturdy/blob/78f51a7a74ebe8adfd055bdbaedfddc05632566f/smart-contracts/ConvexCurveLPVault.sol#L74-L82


# Vulnerability details

Now there are no checks for the amounts to be transferred via _transferYield and _processTreasury. As reward token list is external and an arbitrary token can end up there, in the case when such token doesn't allow for zero amount transfers, the reward retrieval can become unavailable.

I.e. processYield() can be fully blocked for even an extended period, with some low probability, which cannot be controlled otherwise as pool reward token list is external.

Setting the severity to medium as reward gathering is a base functionality for the system and its availability is affected.

## Proof of Concept

_transferYield proceeds with sending the amounts to treasury and yieldManager without checking:

https://github.com/code-423n4/2022-05-sturdy/blob/78f51a7a74ebe8adfd055bdbaedfddc05632566f/smart-contracts/ConvexCurveLPVault.sol#L74-L82

```solidity
    // transfer to treasury
    if (_vaultFee > 0) {
      uint256 treasuryAmount = _processTreasury(_asset, yieldAmount);
      yieldAmount = yieldAmount.sub(treasuryAmount);
    }

    // transfer to yieldManager
    address yieldManager = _addressesProvider.getAddress('YIELD_MANAGER');
    TransferHelper.safeTransfer(_asset, yieldManager, yieldAmount);
```

https://github.com/code-423n4/2022-05-sturdy/blob/78f51a7a74ebe8adfd055bdbaedfddc05632566f/smart-contracts/ConvexCurveLPVault.sol#L205-L209

```solidity
  function _processTreasury(address _asset, uint256 _yieldAmount) internal returns (uint256) {
    uint256 treasuryAmount = _yieldAmount.percentMul(_vaultFee);
    IERC20(_asset).safeTransfer(_treasuryAddress, treasuryAmount);
    return treasuryAmount;
  }
```

The incentive token can be arbitrary. Some ERC20 do not allow zero amounts to be sent:

https://github.com/d-xo/weird-erc20#revert-on-zero-value-transfers

In a situation of such a token added to reward list and zero incentive amount earned the whole processYield call will revert, making reward gathering unavailable until either such token be removed from pool's reward token list or some non-zero reward amount be earned. Both are external processes and aren’t controllable.

## Recommended Mitigation Steps

Consider running the transfers in _transferYield only when yieldAmount is positive:

```solidity
+	if (yieldAmount > 0) {
	    // transfer to treasury
	    if (_vaultFee > 0) {
	      uint256 treasuryAmount = _processTreasury(_asset, yieldAmount);
	      yieldAmount = yieldAmount.sub(treasuryAmount);
	    }

	    // transfer to yieldManager
	    address yieldManager = _addressesProvider.getAddress('YIELD_MANAGER');
	    TransferHelper.safeTransfer(_asset, yieldManager, yieldAmount);
+  }
```

