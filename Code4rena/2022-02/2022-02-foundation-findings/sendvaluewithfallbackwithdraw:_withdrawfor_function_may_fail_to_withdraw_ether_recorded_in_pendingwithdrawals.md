## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [SendValueWithFallbackWithdraw: withdrawFor function may fail to withdraw ether recorded in pendingWithdrawals](https://github.com/code-423n4/2022-02-foundation-findings/issues/12) 

# Lines of code

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/SendValueWithFallbackWithdraw.sol#L37-L77


# Vulnerability details

## Impact
The NFTMarketFees contract and the NFTMarketReserveAuction contract use the _sendValueWithFallbackWithdraw function to send ether to FoundationTreasury, CreatorRecipients, Seller, Bidder.
When the receiver fails to receive due to some reasons (exceeding the gas limit or the receiver contract cannot receive ether), it will record the ether to be sent in the pendingWithdrawals variable. 
```
  function _sendValueWithFallbackWithdraw(
    address payable user,
    uint256 amount,
    uint256 gasLimit
  ) internal {
    if (amount == 0) {
      return;
    }
    // Cap the gas to prevent consuming all available gas to block a tx from completing successfully
    // solhint-disable-next-line avoid-low-level-calls
    (bool success, ) = user.call{ value: amount, gas: gasLimit }("");
    if (!success) {
      // Record failed sends for a withdrawal later
      // Transfers could fail if sent to a multisig with non-trivial receiver logic
      unchecked {
        pendingWithdrawals[user] += amount;
      }
      emit WithdrawPending(user, amount);
    }
  }
```
The user can then withdraw ether via the withdraw or withdrawFor functions.
```
  function withdraw() external {
    withdrawFor(payable(msg.sender));
  }
  function withdrawFor(address payable user) public nonReentrant {
    uint256 amount = pendingWithdrawals[user];
    if (amount == 0) {
      revert SendValueWithFallbackWithdraw_No_Funds_Available();
    }
    pendingWithdrawals[user] = 0;
    user.sendValue(amount);
    emit Withdrawal(user, amount);
  }
```
However, the withdrawFor function can only send ether to the address recorded in pendingWithdrawals. When the recipient is a contract that cannot receive ether, these ethers will be locked in the contract and cannot be withdrawn.
## Proof of Concept
https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/SendValueWithFallbackWithdraw.sol#L37-L77
## Tools Used
None
## Recommended Mitigation Steps
Add the withdrawTo function as follows:

```
  function withdrawTo(address payable to) public nonReentrant {
    uint256 amount = pendingWithdrawals[msg.sneder];
    if (amount == 0) {
      revert SendValueWithFallbackWithdraw_No_Funds_Available();
    }
    pendingWithdrawals[msg.sneder] = 0;
    to.sendValue(amount);
    emit Withdrawal(msg.sneder, amount);
  }
```

