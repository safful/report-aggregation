## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-02-pooltogether-findings/issues/40) 

## Gas Optimizations

### Unchecked math will save on gas
```
  function _computeLockUntil(uint96 _lockDuration) internal view returns (uint96) {
    return uint96(block.timestamp) + _lockDuration; //@audit Gas Optimization Unchecked
  }
```
The `_computeLockUntil` function in `TWABDelegator.sol` can be optimized by adding the `unchecked` directive as this will never overflow the `uint96` type since it is limited by the `MAX_LOCK` constant (which is currently assigned to 180 days() by the call to `_requireLockDuration(_lockDuration);`.
```
justin@Stealth: 18362 » frf test.sol
compiling...
Compiling 2 files with 0.8.12
Compilation finished successfully
success.
Script ran successfully.
Gas Used: 5401
== Logs == 
180 days, 15552000
Current timestamp, 1645633955
Current timestamp + 5000 years, 159325633955
Max uint96, 79228162514264337593543950335

```

### Unneeded Zero Address Check
In the `stake` function of the `TWABDelegator.sol` file, the `_requireRecipientNotZeroAddress` function is called on the `_to` parameter. However, this is unnecessary since the `_mint` function checks for the zero address when called. As such, it would be more gas efficient to not perform this call.
```
 function stake(address _to, uint256 _amount) external {
    _requireRecipientNotZeroAddress(_to);
    //@audit Gas Optimization Above is unneeded since _mint checks whether _to is zero addr or not
    _requireAmountGtZero(_amount);

    IERC20(ticket).safeTransferFrom(msg.sender, address(this), _amount);
    _mint(_to, _amount);
```


### More efficient order of operations in `updateDelegatee`
In the `updateDelegatee` function of the `TWABDelegator.sol` file the `_lockUntil` variable is defined by calling the `_computeLockUntil` function. However, if the `_lockDuration` is 0, then this value is the same as the current `block.timestamp`. As a result, the following code would be an optimization:

Original Code:
```
uint96 _lockUntil = _computeLockUntil(_lockDuration); //@audit Gas Optimization

if (_lockDuration > 0) {
    _delegation.setLockUntil(_lockUntil);
}

_delegateCall(_delegation, _delegatee);

emit DelegateeUpdated(_delegator, _slot, _delegatee, _lockUntil, msg.sender);
```

Optimized Code:
```
uint96 _lockUntil = block.timestamp;
if (_lockDuration > 0) {
    _lockUntil = _computeLockUntil(_lockDuration);
    _delegation.setLockUntil(_lockUntil);
}
_delegateCall(_delegation, _delegatee);
emit DelegateeUpdated(_delegator, _slot, _delegatee, _lockUntil, msg.sender);
```

Here are the tests with the optimizations (* indicates Optimized case):
```
·························|·····························|·············|·············|··············|···············|··············
|  Contract              ·  Method                     ·  Min        ·  Max        ·  Avg         ·  # calls      ·  usd (avg)  │
·························|·····························|·············|·············|··············|···············|··············
|  TWABDelegatorHarness  ·  updateDelegatee            ·     140113  ·     144124  ·      141911  ·            7  ·          -  │
·························|·····························|·············|·············|··············|···············|··············
|  *TWABDelegatorHarness ·  updateDelegatee            ·     139965  ·     144126  ·      141806  ·            7  ·          -  │
·························|·····························|·············|·············|··············|···············|··············
```


## Non-Critcal 

### Missing comment for the `to` parameter
There is no comment on the `to` parameter for the `TransferredDelegation` event in the `TWABDelegator.sol` file.
```
  /**
   * @notice Emitted when a delegator withdraws an amount of tickets from a delegation to their wallet.
   * @param delegator Address of the delegator
   * @param slot  Slot of the delegation
   * @param amount Amount of tickets withdrawn
   */
   //@audit Missing comment for the `to` parameter
  event TransferredDelegation(
    address indexed delegator,
    uint256 indexed slot,
    uint256 amount,
    address indexed to
  );
  ```


  ## Low

  ### Incorrect Event Parameters
  The `TWABDelegator.sol`'s `TicketsStaked` event's first parameter should be the delegator, as such, it should be `msg.sender` not `_to` as `_to` is the recipient.
  ```
    /**
   * @notice Emitted when tickets have been staked.
   * @param delegator Address of the delegator
   * @param amount Amount of tickets staked
   */
  event TicketsStaked(address indexed delegator, uint256 amount);

  ...

    /**
   * @notice Stake `_amount` of tickets in this contract.
   * @dev Tickets can be staked on behalf of a `_to` user.
   * @param _to Address to which the stake will be attributed
   * @param _amount Amount of tickets to stake
   */
  function stake(address _to, uint256 _amount) external {
    _requireRecipientNotZeroAddress(_to); //@audit See here that the _to is the recipient, not the delegator.
    _requireAmountGtZero(_amount);

    IERC20(ticket).safeTransferFrom(msg.sender, address(this), _amount);
    _mint(_to, _amount);

    emit TicketsStaked(_to, _amount);//@audit the first parameter of TicketsStacked should be delegator, not recipient. Should be msg.sender.
  }
  ```

### Incorrect Comment Associated with `transferDelegationTo`
The comments above the `transferDelegationTo` function are incorrect. The first line, which begins with `@notice`, says `The tickets are transferred to the caller`. However, the tickets are transfered to the `_to` parameter as can be seen by the line `_transfer(_delegation, _to, _amount);`
In addition, the comment directly below that states `Will directly send the tickets to the delegator wallet.` This is also incorrect per the above reason.
```
  /**
   * @notice Withdraw an `_amount` of tickets from a delegation. The delegator is assumed to be the caller. The tickets are transferred to the caller.
   * @dev Will directly send the tickets to the delegator wallet.
   * @dev Will revert if delegation is still locked.
   * @param _slot Slot of the delegation
   * @param _amount Amount to withdraw
   * @param _to Account to transfer the withdrawn tickets to
   * @return The address of the Delegation
   */
  function transferDelegationTo(
    uint256 _slot,
    uint256 _amount,
    address _to
  ) external returns (Delegation) {
    _requireRecipientNotZeroAddress(_to);

    Delegation _delegation = Delegation(_computeAddress(msg.sender, _slot));
    _transfer(_delegation, _to, _amount);

    emit TransferredDelegation(msg.sender, _slot, _amount, _to);

    return _delegation;
  }
  ```
  