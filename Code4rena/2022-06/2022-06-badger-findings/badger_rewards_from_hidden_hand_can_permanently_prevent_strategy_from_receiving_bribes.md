## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- valid

# [Badger rewards from Hidden Hand can permanently prevent Strategy from receiving bribes](https://github.com/code-423n4/2022-06-badger-findings/issues/111) 

# Lines of code

https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L428-L430
https://github.com/Badger-Finance/badger-vaults-1.5/blob/3c96bd83e9400671256b235422f63644f1ae3d2a/contracts/BaseStrategy.sol#L351
https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L407-L408


# Vulnerability details

## Impact
If the contract receives rewards from the hidden hand marketplace in BADGER then the contract tries to transfer the same amount of tokens twice to two different accounts, once with `_sendBadgerToTree()` in `MyStrategy` and again with `_processExtraToken()` in the `BasicStrategy` contract. As it is very likely that the strategy will not start with any BADGER tokens, the second transfer will revert (as we are using safeTransfer). This means that `claimBribesFromHiddenHand()` will always revert preventing any other bribes from being received.
## Proof of Concept
1. `claimBribesFromHiddenHand()` is called by strategist
2. Multiple bribes are sent to the strategy including BADGER. For example lets say 50 USDT And 50 BADGER
3. Strategy receives BADGER and calls `_handleRewardTransfer()` which calls `_sendBadgerToTree()`. 50 BADGER is sent to the Badger Tree so balance has dropped to 0.
4. 50 Badger is then again sent to Vault however balance is 0 so the command fails and reverts
5. No more tokens can be claimed anymore

## Tools Used
VS Code
## Recommended Mitigation Steps
`_processExtraToken()` eventually sends the badger to the badger tree through the `Vault` contract. Change
```
    function _sendBadgerToTree(uint256 amount) internal {
        IERC20Upgradeable(BADGER).safeTransfer(BADGER_TREE, amount);
        _processExtraToken(address(BADGER), amount);
    }
```
to
```
    function _sendBadgerToTree(uint256 amount) internal {
        _processExtraToken(address(BADGER), amount);
    }
```

