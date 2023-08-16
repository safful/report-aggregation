## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Owner can steal XDEFI without any capital risk](https://github.com/code-423n4/2022-01-xdefi-findings/issues/52) 

# Handle

onewayfunction


# Vulnerability details

## Impact
The owner of the `XDEFIDistribution` contract can use flash loans to atomically steal XDEFI from the contract without taking on any capital risk.

## Proof of Concept
In my previous submission, "Anyone can steal XDEFI from the `XDEFIDistribution` contract and make the contract insolvent", I showed how any user can use the `onERC721Received` hook of the `_safeMint` function to steal XDEFI tokens from the contract and generally bork the contract's accounting. The attacker in that case took on some risk proportional to the minimum allowable `duration` and was limited in the amount they could steal based on their own capital available (how much XDEFI they had to use during the malicious lockup).

However, when a similar attack is performed by the `owner` of the `XDEFIDistribution` contract, it can be done (1) without the owner taking on any risk at all and (2) the owner can use flashloans to dramatically increase the amount of XDEFI they can steal.

In particular, the owner can perform all of the following in a single transaction (or in a single flashbots bundle):

First, the owner can call the [`setLockPeriods` function](https://github.com/XDeFi-tech/xdefi-distribution/blob/v1.0.0-beta.0/contracts/XDEFIDistribution.sol#L77) to allow `0` duration locks with a large `multiplier`.

Next, they can flash borrow as much XDEFI as possible from DEXs and loaning platforms. Call this amount of XDEFI `X`.

Then they do a (normal) `0` duration lock with `X/2` XDEFI. This could give them a large proportion of locked XDEFI.

Next, they do the "malicious lock" technique that I previously reported, using the remaining `X/2` XDEFI. This means that their first lock with be able to withdraw more than `X/2` XDEFI when they unlock.

Then, in the same transaction -- which is possible because they are using a `0` duration lock -- they can unlock both thier first "normal" lock, as well as their "malicious" lock, giving them more than `X` XDEFI in total.

They can repay the flash loan, and keep the difference.

Since the never have to hold a lock for any positive duration, and never even have to have any exposure to XDEFI, the attack is risk free for them. And since they can use flash loans, they'll likely have access to dramatically more capital than a non-owner (who can't use flash loans) could.


## Recommended Mitigation Steps

In addition to the "use `_mint()` instead of `_safeMint()`" suggestion from the previous submission, I also recommend adding a `require(duration > 0, "INVALID_DURATION");` statement just above [L82](https://github.com/XDeFi-tech/xdefi-distribution/blob/v1.0.0-beta.0/contracts/XDEFIDistribution.sol#L82).

Not only will disallowing `0` duration locks prevent most flashloan shenanigans by the owner, it would also help prevent sandwich attacks that steal incoming distributableX DEFI tokens by sandwiching such incoming txs with a `lock` and `unlock` transaction.

