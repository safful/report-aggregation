## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [first user can steal everyone else's tokens](https://github.com/code-423n4/2022-01-sherlock-findings/issues/39) 

# Handle

egjlmn1


# Vulnerability details

## Impact
A user who joins the systems first (stakes first) can steal everybody's tokens by sending tokens to the system externally.
This attack is possible because you enable staking a small amount of tokens.

## Proof of Concept
See the following attack:
1. the first user (user A) who enters the system stake 1 token
2. another user (user B) is about to stake X tokens
3. user A frontrun and transfer X tokens to the system via `ERC20.transfer`
4. user B stakes X tokens, and the shares he receives is:
`shares = (_amount * totalStakeShares_) / (totalTokenBalanceStakers() - _amount);`
`shares = (X * 1) / (X + 1 + X - X) = X/(X+1) = 0` meaning all the tokens he staked got him no shares, and those tokens are now a part of the single share that user A holds
5. user A can now redeem his shares and get the 1 token he staked, the X tokens user B staked, and the X tokens he `ERC20.transfer` to the system because all the money in the system is in a single share that user A holds.

In general, since there is only a single share, for any user who is going to stake X tokens, if the system has X+1 tokens in its balance, the user won't get any shares and all the money will go to the attacker.

## Tools Used
Manual code review

## Recommended Mitigation Steps
Force users to stake at least some amount in the system (Uniswap forces users to pay at least `1e18`)
That way the amount the attacker will need to ERC20.transfer to the system will be at least `X*1e18` instead of `X` which is unrealistic

