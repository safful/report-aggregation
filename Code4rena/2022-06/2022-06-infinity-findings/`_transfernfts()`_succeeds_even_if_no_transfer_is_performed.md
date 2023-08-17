## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`_transferNFTs()` succeeds even if no transfer is performed](https://github.com/code-423n4/2022-06-infinity-findings/issues/87) 

# Lines of code

https://github.com/code-423n4/2022-06-infinity/blob/765376fa238bbccd8b1e2e12897c91098c7e5ac6/contracts/core/InfinityExchange.sol#L1062


# Vulnerability details

## Impact
If an NFT is sold that does not specify support for the ERC-721 or ERC-1155 standard interface, the sale will still succeed. In doing so, the seller will receive funds from the buyer, but the buyer will not receive any NFT from the seller. This could happen in the following cases:
1. a token that claims to be ERC-721/1155 compliant, but fails to implement the `supportsInterface()` function properly.
2. an NFT that follows a standard other than ERC-721/1155 and does not implement their EIP-165 interfaces.
3. a malicious contract that is deployed to take advantage of this behavior. 

## Proof of Concept
https://gist.github.com/kylriley/3bf0e03d79b3d62dd5a9224ca00c4cb9

## Tools Used
N/A

## Recommended Mitigation Steps
If neither the ERC-721 nor the ERC-1155 interface is supported the function should revert. An alternative approach would be to attempt a `transferFrom` and check the balance before and after to ensure that it succeeded.

