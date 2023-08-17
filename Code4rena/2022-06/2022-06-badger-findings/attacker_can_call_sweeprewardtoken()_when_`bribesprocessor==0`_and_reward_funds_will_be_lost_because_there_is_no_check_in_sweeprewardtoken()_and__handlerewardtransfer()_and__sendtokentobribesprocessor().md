## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- valid

# [attacker can call sweepRewardToken() when `bribesProcessor==0` and reward funds will be lost because there is no check in sweepRewardToken() and _handleRewardTransfer() and _sendTokenToBribesProcessor()](https://github.com/code-423n4/2022-06-badger-findings/issues/18) 

# Lines of code

https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L107-L113
https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L405-L413
https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L421-L425


# Vulnerability details

## Impact
If the value of `bribesProcessor` was `0x0` (the default is `0x0` and `governance()`  can set to `0x0`) then attacker can call `sweepRewardToken()` make contract to send his total balance in attacker specified token to `0x0` address.

## Proof of Concept
the default value of `bribesProcessor` is `0x0` and `governance` can set the value to `0x0` at any time. rewards are stacking in contract address and they are supposed to send to `bribesProcessor`.
This is `sweepRewardToken()` and `_handleRewardTransfer()` and `_sendTokenToBribesProcessor()` code:
```
  /// @dev Function to move rewards that are not protected
  /// @notice Only not protected, moves the whole amount using _handleRewardTransfer
  /// @notice because token paths are hardcoded, this function is safe to be called by anyone
  /// @notice Will not notify the BRIBES_PROCESSOR as this could be triggered outside bribes
  function sweepRewardToken(address token) public nonReentrant {
      _onlyGovernanceOrStrategist();
      _onlyNotProtectedTokens(token);

      uint256 toSend = IERC20Upgradeable(token).balanceOf(address(this));
      _handleRewardTransfer(token, toSend);
  }

  function _handleRewardTransfer(address token, uint256 amount) internal {
      // NOTE: BADGER is emitted through the tree
      if (token == BADGER) {
          _sendBadgerToTree(amount);
      } else {
          // NOTE: All other tokens are sent to bribes processor
          _sendTokenToBribesProcessor(token, amount);
      }
  }

  function _sendTokenToBribesProcessor(address token, uint256 amount) internal {
      // TODO: Too many SLOADs
      IERC20Upgradeable(token).safeTransfer(address(bribesProcessor), amount);
      emit RewardsCollected(token, amount);
  }
```
As you can see calling `sweepRewardToken()` eventually (`sweepRewardToken() -> _handleRewardTransfer() -> _sendTokenToBribesProcessor()`) would transfer reward funds to `bribesProcessor` and there is no check that `bribesProcessor!=0x0` in execution follow. so attacker can call `sweepRewardToken()` when `bribesProcessor` is `0x0` and contract will lose all reward tokens.


## Tools Used
VIM

## Recommended Mitigation Steps
check the value of `bribesProcessor` in `_sendTokenToBribesProcessor()`

