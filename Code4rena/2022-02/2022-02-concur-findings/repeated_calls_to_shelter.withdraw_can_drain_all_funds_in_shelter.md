## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Repeated Calls to Shelter.withdraw Can Drain All Funds in Shelter](https://github.com/code-423n4/2022-02-concur-findings/issues/246) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/Shelter.sol#L52-L57


# Vulnerability details

## Impact
tl;dr Anyone who can call `withdraw` to withdraw their own funds can call it repeatedly to withdraw the funds of others. `withdraw` should only succeed if the user hasn't withdrawn the token already.

The shelter can be used for users to withdraw funds in the event of an emergency. The `withdraw` function allows callers to withdraw tokens based on the tokens they have deposited into the shelter client: ConvexStakingWrapper. However, `withdraw` does not check if a user has already withdrawn their tokens. Thus a user that can `withdraw` tokens, can call withdraw repeatedly to steal the tokens of others.

## Proof of Concept

tl;dr an attacker that can successfully call `withdraw` once on a shelter, can call it repeatedly to steal the funds of others. Below is a detailed scenario where this situation can be exploited.

1. Mallory deposits 1 `wETH` into `ConvexStakingWrapper` using [`deposit`](https://github.com/code-423n4/2022-02-concur/blob/shelter-client/contracts/ConvexStakingWrapper.sol#L280). Let's also assume that other users have deposited 2 `wETH` into the same contract.
2. An emergency happens and the owner of `ConvexStakingWrapper` calls `setShelter(shelter)` and `enterShelter([pidOfWETHToken, ...])`. Now `shelter` has 3 `wETH` and is activated for `wETH`.
3. Mallory calls `shelter.withdraw(wETHAddr, MalloryAddr)`, mallory will rightfully receive 1 wETH because her share of wETH in the shelter is 1/3.
4. Mallory calls `shelter.withdraw(wETHAddr, MalloryAddr)` again, receiving 1/3*2 = 2/3 wETH. `withdraw` does not check that she has already withdrawn. This time, the wETH does not belong to her, she has stolen the wETH of the other users. She can continue calling `withdraw` to steal the rest of the funds


## Tools Used

Manual inspection.

## Recommended Mitigation Steps

To mitigate this, `withdraw` must first check that `msg.sender` has not withdrawn this token before and `withdraw` must also record that `msg.sender` has withdrawn the token.
The exact steps for this are below:
1. Add the following line to the beginning of `withdraw` (line 53):
```
require(!claimed[_token][msg.sender], "already claimed")
```
2.  Replace [line 55](https://github.com/code-423n4/2022-02-concur/blob/main/contracts/Shelter.sol#L55) with the following:
```
claimed[_token][msg.sender] = true;
```
This replacement is necessary because we want to record who is withdrawing, not where they are sending the token which isn't really useful info.

