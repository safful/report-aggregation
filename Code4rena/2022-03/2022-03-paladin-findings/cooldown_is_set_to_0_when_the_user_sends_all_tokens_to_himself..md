## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [cooldown is set to 0 when the user sends all tokens to himself.](https://github.com/code-423n4/2022-03-paladin-findings/issues/8) 

# Lines of code

https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L891-L905


# Vulnerability details

## Impact
In the _beforeTokenTransfer function, cooldowns will be set to 0 when the user transfers all tokens to himself.
Consider the following scenario
Day 0: The user stakes 100 tokens and calls the cooldown function
Day 10: the user wanted to unstake the tokens, but accidentally transferred all the tokens to himself, which caused the cooldown to be set to 0 and the user could not unstake.
## Proof of Concept
https://github.com/code-423n4/2022-03-paladin/blob/main/contracts/HolyPaladinToken.sol#L891-L905
## Tools Used
None
## Recommended Mitigation Steps
```
  function _beforeTokenTransfer(
      address from,
      address to,
      uint256 amount
  ) internal virtual override {
      if(from != address(0)) { //check must be skipped on minting
          // Only allow the balance that is unlocked to be transfered
          require(amount <= _availableBalanceOf(from), "hPAL: Available balance too low");
      }

      // Update user rewards before any change on their balance (staked and locked)
      _updateUserRewards(from);

      uint256 fromCooldown = cooldowns[from]; //If from is address 0x00...0, cooldown is always 0

      if(from != to) {
          // Update user rewards before any change on their balance (staked and locked)
          _updateUserRewards(to);
          // => we don't want a self-transfer to double count new claimable rewards
          // + no need to update the cooldown on a self-transfer

          uint256 previousToBalance = balanceOf(to);
          cooldowns[to] = _getNewReceiverCooldown(fromCooldown, amount, to, previousToBalance);
          // If from transfer all of its balance, reset the cooldown to 0
          uint256 previousFromBalance = balanceOf(from);
          if(previousFromBalance == amount && fromCooldown != 0) {
              cooldowns[from] = 0;
          }
      }
  }
```

