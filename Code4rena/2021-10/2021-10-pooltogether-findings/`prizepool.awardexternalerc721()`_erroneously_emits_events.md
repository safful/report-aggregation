## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- resolved

# [`PrizePool.awardExternalERC721()` Erroneously Emits Events](https://github.com/code-423n4/2021-10-pooltogether-findings/issues/62) 

# Handle

leastwood


# Vulnerability details

## Impact

The `awardExternalERC721()` function uses solidity's try and catch statement to ensure a single tokenId cannot deny function execution. If the try statement fails, an `ErrorAwardingExternalERC721` event is emitted with the relevant error, however, the failed tokenId is not removed from the list of tokenIds emitted at the end of function execution. As a result, the `AwardedExternalERC721` is emitted with the entire list of tokenIds, regardless of failure.  An off-chain script or user could therefore be tricked into thinking an ERC721 tokenId was successfully awarded.

## Proof of Concept

https://github.com/pooltogether/v4-core/blob/master/contracts/prize-pool/PrizePool.sol#L250-L270

## Tools Used

Manual code review

## Recommended Mitigation Steps

Consider emitting only successfully transferred tokenIds in the `AwardedExternalERC721` event.

