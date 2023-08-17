## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- M-01

# [Bypass userWithdrawLimitPerPeriod check](https://github.com/code-423n4/2022-12-prepo-findings/issues/49) 

# Lines of code

https://github.com/prepo-io/prepo-monorepo/blob/feat/2022-12-prepo/apps/smart-contracts/core/contracts/WithdrawHook.sol#L70


# Vulnerability details

## Impact
User can bypass the `userWithdrawLimitPerPeriod` check by transferring balance to another account

## Proof of Concept
1. Assume `userWithdrawLimitPerPeriod` is set to `1000`
2. User A has current deposit of amount `2000` and wants to withdraw everything instantly
3. User A calls the withdraw function and takes out the `1000` amount

```
function withdraw(uint256 _amount) external override nonReentrant {
    uint256 _baseTokenAmount = (_amount * baseTokenDenominator) / 1e18;
    uint256 _fee = (_baseTokenAmount * withdrawFee) / FEE_DENOMINATOR;
    if (withdrawFee > 0) { require(_fee > 0, "fee = 0"); }
    else { require(_baseTokenAmount > 0, "amount = 0"); }
    _burn(msg.sender, _amount);
    uint256 _baseTokenAmountAfterFee = _baseTokenAmount - _fee;
    if (address(withdrawHook) != address(0)) {
      baseToken.approve(address(withdrawHook), _fee);
      withdrawHook.hook(msg.sender, _baseTokenAmount, _baseTokenAmountAfterFee);
      baseToken.approve(address(withdrawHook), 0);
    }
    baseToken.transfer(msg.sender, _baseTokenAmountAfterFee);
    emit Withdraw(msg.sender, _baseTokenAmountAfterFee, _fee);
  }
```

4. Remaining `1000` amount cannot be withdrawn since `userWithdrawLimitPerPeriod` is reached

```
function hook(
    address _sender,
    uint256 _amountBeforeFee,
    uint256 _amountAfterFee
  ) external override onlyCollateral {
...
require(userToAmountWithdrawnThisPeriod[_sender] + _amountBeforeFee <= userWithdrawLimitPerPeriod, "user withdraw limit exceeded");
...
}
```

5. User simply transfer his balance to his another account and withdraw from that account

6. Since withdraw limit is tied to account so this new account will be allowed to make withdrawal thus bypassing userWithdrawLimitPerPeriod


## Recommended Mitigation Steps
User should only be allowed to transfer leftover limit. For example if User already utilized limit X then he should only be able to transfer userWithdrawLimitPerPeriod-X