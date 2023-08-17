## Tags

- bug
- 3 (High Risk)
- high quality report
- primary issue
- sponsor confirmed
- upgraded by judge
- selected for report
- H-03

# [Outstanding reserved tokens are incorrectly counted in total redemption weight](https://github.com/code-423n4/2022-10-juicebox-findings/issues/129) 

# Lines of code

https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/JBTiered721DelegateStore.sol#L563-L566


# Vulnerability details

## Impact
The amounts redeemed in overflow redemption can be calculated incorrectly due to incorrect accounting of the outstanding number of reserved tokens.
## Proof of Concept
Project contributors are allowed to redeem their NFT tokens for a portion of the overflow (excessive funded amounts). The amount a contributor receives is calculated as [overflow * (user's redemption rate / total redemption weight)](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/abstract/JB721Delegate.sol#L135-L142), where user's redemption weight is [the total contribution floor of all their NFTs](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/JBTiered721DelegateStore.sol#L532-L539) and total redemption weight is [the total contribution floor of all minted NFTs](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/JBTiered721DelegateStore.sol#L563-L566). Since the total redemption weight is the sum of individual contributor redemption weights, the amount they can redeem is proportional to their contribution.

However, the total redemption weight calculation incorrectly accounts outstanding reserved tokens ([JBTiered721DelegateStore.sol#L563-L566](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/JBTiered721DelegateStore.sol#L563-L566)):
```solidity
// Add the tier's contribution floor multiplied by the quantity minted.
weight +=
  (_storedTier.contributionFloor *
    (_storedTier.initialQuantity - _storedTier.remainingQuantity)) +
  _numberOfReservedTokensOutstandingFor(_nft, _i, _storedTier);
```
Specifically, the *number* of reserved tokens is added to the *weight* of minted tokens. This disrupts the redemption amount calculation formula since the total redemption weight is in fact not the sum of individual contributor redemption weights.
## Tools Used
Manual review
## Recommended Mitigation Steps
Two options can be seen:
1. if the outstanding number of reserved tokens is considered minted (which seems to be so, judging by [this logic](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/JBTiered721DelegateStore.sol#L1058-L1063)) then it needs to be added to the quantity, i.e.:
    ```diff
    --- a/contracts/JBTiered721DelegateStore.sol
    +++ b/contracts/JBTiered721DelegateStore.sol
    @@ -562,8 +562,7 @@ contract JBTiered721DelegateStore is IJBTiered721DelegateStore {
          // Add the tier's contribution floor multiplied by the quantity minted.
          weight +=
            (_storedTier.contributionFloor *
    -          (_storedTier.initialQuantity - _storedTier.remainingQuantity)) +
    -        _numberOfReservedTokensOutstandingFor(_nft, _i, _storedTier);
    +          (_storedTier.initialQuantity - _storedTier.remainingQuantity +
    +           _numberOfReservedTokensOutstandingFor(_nft, _i, _storedTier)));

          unchecked {
            ++_i;
    ```
1. if it's not considered minted, then it shouldn't be counted at all.