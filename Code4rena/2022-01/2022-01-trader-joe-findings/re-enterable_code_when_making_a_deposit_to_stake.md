## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Re-enterable Code When Making a Deposit to Stake](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/127) 

# Handle

kirk-baird


# Vulnerability details

## Impact

Note: this attack requires `rJoe` to relinquish control during `tranfer()` which under the current [RocketJoeToken](https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/RocketJoeToken.sol) it does not. Thus this vulnerability is raised as medium rather than high. Although it's not exploitable currently, it is a highly risky code pattern that should be avoided.

This vulnerability would allow the entire rJoe balance to be drained from the contract.

## Proof of Concept

The function [deposit()](https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/RocketJoeStaking.sol#L96) would be vulnerable to reentrancy if rJoe relinquished control flow.

The following lines show the reward calculations in variable `pending`. These calculations use two state variables `user.amount` and `user.rewardDebt`. Each of these are updated after `_safeRJoeTransfer()`.

Thus if an attacker was able to get control flow during the `rJoe::tranfer()` function they would be able to reenter `deposit()` and the value calculated for `pending`would be the same as the previous iteration hence they would again be transferred `pending` rJoe tokens. During the rJoe transfer the would again gain control of the execution and call `deposit()` again. The process could be repeated until the entire rJoe balance of the contract has been transferred to the attacker.

```solidity
        if (user.amount > 0) {
            uint256 pending = (user.amount * accRJoePerShare) /
                PRECISION -
                user.rewardDebt;
            _safeRJoeTransfer(msg.sender, pending);
        }
        user.amount = user.amount + _amount;
        user.rewardDebt = (user.amount * accRJoePerShare) / PRECISION;
```


## Tools Used

n/a

## Recommended Mitigation Steps

There are two possible mitigations. First is to use the [openzeppelin reentrancy guard](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/ReentrancyGuard.sol) over the `deposit()` function which will prevent multiple deposits being made simultaneously.

The second mitigation is to follow the [checks-effects-interactions](https://docs.soliditylang.org/en/v0.8.11/security-considerations.html#re-entrancy) pattern. This would involve updating all state variables before making any external calls.

