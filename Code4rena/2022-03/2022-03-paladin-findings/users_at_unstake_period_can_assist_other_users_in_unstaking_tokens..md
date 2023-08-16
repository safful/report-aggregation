## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Users at UNSTAKE_PERIOD can assist other users in unstaking tokens.](https://github.com/code-423n4/2022-03-paladin-findings/issues/7) 

# Lines of code

https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L1131


# Vulnerability details

## Impact
Consider the following scenario:
Day 0: User A stakes 200 tokens and calls the cooldown function. At this time, user A's cooldown is Day 0.
Day 15: User B stakes 100 tokens, but then wants to unstake tokens. So user A said that he could assist user B in unstaking tokens, and this could be done by deploying a smart contract.
In the smart contract deployed by user A, user B first needs to transfer 100 tokens to user A. In the _getNewReceiverCooldown function, _senderCooldown is Day 15 and receiverCooldown is Day 0, so the latest cooldown of user A is (100 * Day 15 + 200 * Day 0)/(100+200) = Day 5.
```
    function _getNewReceiverCooldown(
        uint256 senderCooldown,
        uint256 amount,
        address receiver,
        uint256 receiverBalance
    ) internal view returns(uint256) {
        uint256 receiverCooldown = cooldowns[receiver];

        // If receiver has no cooldown, no need to set a new one
        if(receiverCooldown == 0) return 0;

        uint256 minValidCooldown = block.timestamp - (COOLDOWN_PERIOD + UNSTAKE_PERIOD);

        // If last receiver cooldown is expired, set it back to 0
        if(receiverCooldown < minValidCooldown) return 0;

        // In case the given senderCooldown is 0 (sender has no cooldown, or minting)
        uint256 _senderCooldown = senderCooldown < minValidCooldown ? block.timestamp : senderCooldown;

        // If the sender cooldown is better, we keep the receiver cooldown
        if(_senderCooldown < receiverCooldown) return receiverCooldown;

        // Default new cooldown, weighted average based on the amount and the previous balance
        return ((amount * _senderCooldown) + (receiverBalance * receiverCooldown)) / (amount + receiverBalance);

    }
```
Since User A is still at UNSTAKE_PERIOD after receiving the tokens, User A unstakes 100 tokens and sends it to User B.

After calculation, we found that when user A has a balance of X and is at the edge of UNSTAKE_PERIOD, user A can assist in unstaking the X/2 amount of tokens just staked.

## Proof of Concept
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L1131

## Tools Used
None
## Recommended Mitigation Steps
After calculation, we found that the number of tokens that users at the edge of UNSTAKE_PERIOD can assist in unstaking conforms to the following equation
UNSTAKE_PERIOD/COOLDOWN_PERIOD = UNSTAKE_AMOUNT/USER_BALANCE, when COOLDOWN_PERIOD remains unchanged, the smaller the UNSTAKE_PERIOD, the less tokens the user can assist in unstaking, so UNSTAKE_PERIOD can be adjusted to alleviate this situation.

