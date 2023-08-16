## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Duplicate NFTs Can Be Minted if `payableToken` Has a Callback Attached to it](https://github.com/code-423n4/2022-03-joyn-findings/issues/121) 

# Lines of code

https://github.com/code-423n4/2022-03-joyn/blob/main/core-contracts/contracts/CoreCollection.sol#L139-L167
https://github.com/code-423n4/2022-03-joyn/blob/main/core-contracts/contracts/ERC721Payable.sol#L50-L56


# Vulnerability details

## Impact

The `mintToken()` function is called to mint unique tokens from an `ERC721` collection. This function will either require users to provide a merkle proof to claim an airdropped token or pay a fee in the form of a `payableToken`. However, because the `payableToken` is paid before a token is minted, it may be possible to reenter the `mintToken()` function if there is a callback attached before or after the token transfer. Because `totalSupply()` has not been updated for the new token, a user is able to bypass the `totalSupply() + amount <= maxSupply` check. As a result, if the user mints the last token, they can reenter and mint duplicate NFTs as the way `tokenId` is generated will wrap around to the start again.

## Proof of Concept

For the sake of this example, let's say `startingIndex = 0` and `maxSupply = 100`. `tokenId` is minted according to `((startingIndex + totalSupply()) % maxSupply) + 1`. If we see that a user mints a token where `totalSupply() = maxSupply - 1 = 99` and they reenter the function, then the next token to mint will actually be of index `1` as `totalSupply() % maxSupply = 0`. Calculating the first `tokenId`, we get `((0 + 0) % maxSupply) + 1 = 1` which is a duplicate of our example.

## Recommended Mitigation Steps

Consider adding reentrancy protections to prevent users from abusing this behaviour. It may also be useful to follow the checks-effects pattern such that all external/state changing calls are made at the end.

