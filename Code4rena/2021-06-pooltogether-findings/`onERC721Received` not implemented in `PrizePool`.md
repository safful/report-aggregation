## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`onERC721Received` not implemented in `PrizePool`](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/118) 

# Handle

shw


# Vulnerability details

## Impact

The `PrizePool` contract does not implement the `onERC721Received` function, which is considered a best practice to transfer ERC721 tokens from contracts to contracts. The absence of this function could prevent `PrizePool` from receiving ERC721 tokens from other contracts via `safeTransferFrom`.

## Proof of Concept

Referenced code:
[PrizePool.sol](https://github.com/code-423n4/2021-06-pooltogether/blob/main/contracts/PrizePool.sol)

## Recommended Mitigation Steps

Consider adding an implementation of the `onERC721Received` function in `PrizePool`.

