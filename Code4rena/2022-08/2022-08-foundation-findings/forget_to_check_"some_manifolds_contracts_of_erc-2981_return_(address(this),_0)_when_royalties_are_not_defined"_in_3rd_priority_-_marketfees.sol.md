## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- old-submission-method

# [Forget to check "Some manifolds contracts of ERC-2981 return (address(this), 0) when royalties are not defined" in 3rd priority - MarketFees.sol](https://github.com/code-423n4/2022-08-foundation-findings/issues/147) 

# Lines of code

https://github.com/code-423n4/2022-08-foundation/blob/792e00df429b0df9ee5d909a0a5a6e72bd07cf79/contracts/mixins/shared/MarketFees.sol#L299-L301


# Vulnerability details

## Impact
Wrong return of `cretorShares` and `creatorRecipients` can make real royalties party can't gain the revenue of sale. 
 
## Proof of concept 
Function `getFees()` firstly [call](https://github.com/code-423n4/2022-08-foundation/blob/792e00df429b0df9ee5d909a0a5a6e72bd07cf79/contracts/mixins/shared/MarketFees.sol#L422-L430) to function `internalGetImmutableRoyalties` to get the list of `creatorRecipients` and `creatorShares` if the `nftContract` define ERC2981 royalties.

```solidity=
try implementationAddress.internalGetImmutableRoyalties(nftContract, tokenId) returns (
  address payable[] memory _recipients,
  uint256[] memory _splitPerRecipientInBasisPoints
) {
  (creatorRecipients, creatorShares) = (_recipients, _splitPerRecipientInBasisPoints);
} catch // solhint-disable-next-line no-empty-blocks
{
  // Fall through
}
```
-----
In the [1st priority](https://github.com/code-423n4/2022-08-foundation/blob/792e00df429b0df9ee5d909a0a5a6e72bd07cf79/contracts/mixins/shared/MarketFees.sol#L236-L255) it check the `nftContract` define the function `royaltyInfo` or not. If yes, it get the return value `receiver` and `royaltyAmount`. In some manifold contracts of erc2981, it `return (address(this), 0)` when royalties are not defined. So we ignore it when the `royaltyAmount = 0` 
```solidity=
  try IRoyaltyInfo(nftContract).royaltyInfo{ gas: READ_ONLY_GAS_LIMIT }(tokenId, BASIS_POINTS) returns (
    address receiver,
    uint256 royaltyAmount
  ) {
    // Manifold contracts return (address(this), 0) when royalties are not defined
    // - so ignore results when the amount is 0
    if (royaltyAmount > 0) {
      recipients = new address payable[](1);
      recipients[0] = payable(receiver);
      splitPerRecipientInBasisPoints = new uint256[](1);
      // The split amount is assumed to be 100% when only 1 recipient is returned
      return (recipients, splitPerRecipientInBasisPoints);
    }
```
----
In the same sense, the [3rd priority](https://github.com/code-423n4/2022-08-foundation/blob/792e00df429b0df9ee5d909a0a5a6e72bd07cf79/contracts/mixins/shared/MarketFees.sol#L297-L312) (it can reach to 3rd priority when function `internalGetImmutableRoyalies` fail to return some royalties) should check same as the 1st priority with the `royaltyRegistry.getRoyaltyLookupAddress`. But the 3rd priority forget to check the case when `royaltyAmount == 0`. 
```solidity=
  try IRoyaltyInfo(nftContract).royaltyInfo{ gas: READ_ONLY_GAS_LIMIT }(tokenId, BASIS_POINTS) returns (
    address receiver,
    uint256 /* royaltyAmount */
  ) {
    recipients = new address payable[](1);
    recipients[0] = payable(receiver);
    splitPerRecipientInBasisPoints = new uint256[](1);
    // The split amount is assumed to be 100% when only 1 recipient is returned
    return (recipients, splitPerRecipientInBasisPoints);
  } 
```
It will make [function](https://github.com/code-423n4/2022-08-foundation/blob/792e00df429b0df9ee5d909a0a5a6e72bd07cf79/contracts/mixins/shared/MarketFees.sol#L98) `_distributeFunds()` transfer to wrong `creatorRecipients` (for example erc2981 return `(address(this), 0)`, market will transfer creator revenue to `address(this)` - market contract, and make the fund freeze in contract forever).

This case just happen when
* `nftContract` doesn't have any support for royalties info 
* `overrideContract` which was fetched from`royaltyRegistry.getRoyaltyLookupAddress(nftContract)` implements both function `getRoyalties` and `royaltyInfo` but doesn't support `royaltyInfo` by returning `(address(this), 0)`. 

## Tools Used
Manual review 
  
## Recommended Mitigation Steps
Add check if `royaltyAmount > 0` or not in 3rd priority

