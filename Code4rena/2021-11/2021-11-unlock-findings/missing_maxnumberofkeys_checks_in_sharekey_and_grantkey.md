## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Missing maxNumberOfKeys checks in shareKey and grantKey](https://github.com/code-423n4/2021-11-unlock-findings/issues/55) 

# Handle

kenzo


# Vulnerability details

More keys can be minted than maxNumberOfKeys since `shareKey` and `grantKey` do not check if the lock is sold out.

## Impact
More keys can be minted than intended.

## Proof of Concept
In both `shareKey` and `grantKey`, if minting a new token, a new token is simply minted (and `_totalSupply` increased) without checking it against `maxNumberOfKeys`.
This is unlike `purchase`, which has the `notSoldOut` modifier.
`grantKey`:
https://github.com/code-423n4/2021-11-unlock/blob/main/smart-contracts/contracts/mixins/MixinGrantKeys.sol#L41:#L42
`shareKey`:
https://github.com/code-423n4/2021-11-unlock/blob/main/smart-contracts/contracts/mixins/MixinTransfer.sol#L83:#L84
Both functions call `_assignNewTokenId` which does not check maxNumberOfKeys. 
https://github.com/code-423n4/2021-11-unlock/blob/main/smart-contracts/contracts/mixins/MixinKeys.sol#L311:#L322
So you can say that `_assignNewTokenId` is actually the root of the error, and this is why I am submitting this as 1 finding and not 2 (for grantKey/shareKey).

## Recommended Mitigation Steps
Add a check to `_assignNewTokenId` that will revert if we need to record a new key and `maxNumberOfKeys` has been reached.

