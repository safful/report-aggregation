## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Withdrawal delay can be circumvented](https://github.com/code-423n4/2022-03-prepo-findings/issues/54) 

# Lines of code

https://github.com/code-423n4/2022-03-prepo/blob/f63584133a0329781609e3f14c3004c1ca293e71/contracts/core/Collateral.sol#L97


# Vulnerability details

## Impact
After initiating a withdrawal with `initiateWithdrawal`, it's still possible to transfer the collateral tokens.
This can be used to create a second account, transfer the accounts to them and initiate withdrawals at a different time frame such that one of the accounts is always in a valid withdrawal window, no matter what time it is.
If the token owner now wants to withdraw they just transfer the funds to the account that is currently in a valid withdrawal window.

Also, note that each account can withdraw the specified `amount`. Creating several accounts and circling & initiating withdrawals with all of them allows withdrawing larger amounts **even at the same block** as they are purchased in the future.

I consider this high severity because it breaks core functionality of the Collateral token.

#### POC
For example, assume the `_delayedWithdrawalExpiry = 20` blocks. Account A owns 1000 collateral tokens, they create a second account B.
- At `block=0`, A calls `initiateWithdrawal(1000)`. They send their balance to account B.
- At `block=10`, B calls `initiateWithdrawal(1000)`. They send their balance to account A.
- They repeat these steps, alternating the withdrawal initiation every 10 blocks.
- One of the accounts is always in a valid withdrawal window (`initiationBlock < block && block <= initiationBlock + 20`). They can withdraw their funds at any time.

## Recommended Mitigation Steps
If there's a withdrawal request for the token owner (`_accountToWithdrawalRequest[owner].blockNumber > 0`), disable their transfers for the time.

```solidity
// pseudo-code not tested
beforeTransfer(from, to, amount) {
  super();
  uint256 withdrawalStart =  _accountToWithdrawalRequest[from].blockNumber;
  if(withdrawalStart > 0 && withdrawalStart + _delayedWithdrawalExpiry < block.number) {
    revert(); // still in withdrawal window
  }
}
```

