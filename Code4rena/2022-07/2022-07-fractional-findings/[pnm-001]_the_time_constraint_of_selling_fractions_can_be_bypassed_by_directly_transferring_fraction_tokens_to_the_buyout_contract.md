## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [[PNM-001] The time constraint of selling fractions can be bypassed by directly transferring fraction tokens to the buyout contract](https://github.com/code-423n4/2022-07-fractional-findings/issues/283) 

# Lines of code

https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Buyout.sol#L206


# Vulnerability details

### Description

The `end` function in the `Buyout` contract uses `IERC1155(token).balanceOf(address(this), id)` to determine the amount of deposited fraction tokens without distinguishing whether those fraction tokens are depositied by the `sellFractions` function or by direct transferring. Note that only the `sellFractions` function is constrained by `PROPOSAL_PERIOD`.

This vulnerability lets a 51-holder gain the whole batch of NFTs without paying for the rest 49\% fractions.

Assume a vault X creates 100 fraction tokens and the market-decided price of a fraction token is 1 ether (i.e., the ideal value of the locked NFTs in vault X is 100 ether). Let's also assume that Alice holds 51 tokens (maybe by paying 51 ether on opensea).

Followings are two scenarios, where the benign one follows the normal workflow and the malicious one exploits the vulnerability.

### Benign Scenario

+ Alice starts a buyout by depositing her 51 fraction tokens and 49 ether, making the `fractionPrice` 1 ether
+ Other users are satisfied with the provided price, and hence no one buys or sells their fraction tokens
+ The buyout succeeds:
    + Alice gets the locked NFTs
    + Other fraction holders can invoke `cash` to redeem their fraction tokens with a price of 1 ether
+ As a result, Alice paid 100 ether in total to get the locked NFTs. 

### Malicious Scenario

+ Alice starts a buyout by depositing 0 fraction tokens and 1 wei, making the `fractionPrice` 0.01 wei.
    + Note that Alice can create a separated account whose balance for the fraction token is 0, to start the buyout
+ No one is satisfied with the price (0.01 wei v/s 1 ether) and hence they will try to buy fraction tokens to reject the buyout
    + Since there is not any fraction tokens locked in the `Buyout` contract from Alice, other users do not need to do anything
+ Alice invokes the `end` function
    + But before invoking the `end` function, __Alice directly invokes `IERC1155(token).safeTransferFrom` to send the rest 51 fraction token to the `Buyout` contract__
    + The `end` function will treat the buyout successful, since the `IERC1155(token).balanceOf(address(this), id)` is bigger than 50\%
    + The above two message calls happen in a single transaction, hence no one can front-run
+ As a result
    + __Alice only paid 51 ether to get the locked NFTs whose value is 100 ether__
    + __Other fraction holders get nothing (but they had paid for the fraction token before)__

In short, a malicious users can buy any NFT by just paying half of the NFT's market price

### Suggested Fix

For each buyout, add a new field to record the amount of fraction tokens deposited by `sellFractions`. And in the `end` function, use the newly-added field to determine whether the buyout can be processed or not.

