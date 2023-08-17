## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Possible token reentrancy in release() of BathBuddy.sol](https://github.com/code-423n4/2022-05-rubicon-findings/issues/283) 

# Lines of code

https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/peripheral_contracts/BathBuddy.sol#L114


# Vulnerability details

## Impact
If a token with callback capabilities is used as a token to vested, then a malicious beneficiary may get the vested amount back without waiting for the vesting period.

## Proof of Concept

In the function release, line (<https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/peripheral_contracts/BathBuddy.sol#L87>), there’s no modifier to stop reentrancy, in the other contracts it would be the synchronized modifier. If a token could reenter with a hook in a malicious contract (an ERC777 token, for example, which is backwards compatible with ERC20), released token counter array (<https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/peripheral_contracts/BathBuddy.sol#L116>) wouldn’t be updated, enabling the withdrawal of the vested amount before the vesting period ends. A plausible scenario would be:

1) A malicious beneficiary contract B calls the release() function with itself as the recipient, everything goes according to the function, and transfer and callback to the malicious beneficiary contract happens.

2) Contract B contains tokensReceived(), a function in the ERC777 token that allows for callback to the victim contract as you can see here https://twitter.com/transmissions11/status/1496944873760428058/ (This function also can be any function that is analogous to a fallback function that might be implemented in a modified ERC20. As it can be seen, any token that would give the attacker control over the execution flow will suffice.)

3) Inside the tokensReceived() function, a call is made back to the release function.

5) This steps are repeated until vested amount is taken back.

4) This allows for the malicious beneficiary contract to redeem the vested amount while bypassing the vesting period, due to the released token counter array (https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/peripheral_contracts/BathBuddy.sol#L116) which controls how many tokens are released (https://github.com/code-423n4/2022-05-rubicon/blob/8c312a63a91193c6a192a9aab44ff980fbfd7741/contracts/peripheral_contracts/BathBuddy.sol#L101) being updated only after the transferring of all tokens occurs. As this is the case, malicious beneficiary  can get the usual amount that they could withdraw at the time indefinite amount of times (as result of released in line 101 will be 0), thus approximately getting all of their vested amount back without waiting for the vesting period. (fees not included). 


There's also precedents of similar bugs that reported, as seen here:
https://github.com/code-423n4/2022-01-behodler-findings/issues/154#issuecomment-1029448627


## Tools used
Manual code review, talks with dev

## Recommended Mitigations Steps
1) Consider adding a mutex such as nonReentrant, or the synchronized modifier used in the other contracts.

2) Implement checks-effects-interactions pattern.

