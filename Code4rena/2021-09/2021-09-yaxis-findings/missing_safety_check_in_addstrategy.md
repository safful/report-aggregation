## Tags

- bug
- 1 (Low Risk)
- sponsor acknowledged
- sponsor confirmed

# [missing safety check in addStrategy](https://github.com/code-423n4/2021-09-yaxis-findings/issues/23) 

# Handle

jonah1005


# Vulnerability details

## Impact
There's no safety check in controller's `addStrategy`. When the same strategy is added to a vault twice, the protocol breakdowns in several ways. 

1. Removing that strategy would always raise errors.
2. `_vaultDetails[_vault].balances[_strategy]` would not track strategy's balance correctly; `getBestStrategyWithdraw` would have a wrong answer and makes withdrawing from the strategy to raise error in certain scenarios.

I consider this a low-risk issue.

## Proof of Concept
This is the web3.py script:
```python
controller.functions.addStrategy(vault.address, strategy.address, cap, 0).transact()
controller.functions.addStrategy(vault.address, strategy.address, cap, 0).transact()

# would not be able to removestrategy
controller.functions.removeStrategy(vault.address, strategy.address, 0).transact()
```

## Tools Used
Hardhat

## Recommended Mitigation Steps

The controller should raise an error if the strategy has been added to the protocol(any vault). As adding the same strategy to two different vaults would have worse results, the controller can maintain a map to record each strategy's status.

