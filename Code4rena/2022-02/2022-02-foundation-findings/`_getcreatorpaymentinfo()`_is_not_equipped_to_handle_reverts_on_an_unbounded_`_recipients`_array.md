## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`_getCreatorPaymentInfo()` is Not Equipped to Handle Reverts on an Unbounded `_recipients` Array](https://github.com/code-423n4/2022-02-foundation-findings/issues/85) 

# Lines of code

https://github.com/code-423n4/2022-02-foundation/blob/main/contracts/mixins/NFTMarketCreators.sol#L49-L251


# Vulnerability details

## Impact

The `_getCreatorPaymentInfo()` function is utilised by `_distributeFunds()` whenever an NFT sale is made. The function uses `try` and `catch` statements to handle bad API endpoints. As such, a revert in this function would lead to NFTs that are locked in the contract. Some API endpoints receive an array of recipient addresses which are iterated over. If for whatever reason the function reverts inside of a `try` statement, the revert is actually not handled and it will not fall through to the empty `catch` statement.

## Proof of Concept

The end result is that valid and honest NFT contracts may revert if the call runs out of gas due to an unbounded `_recipients` array. `try` statements are only able to handle external calls.

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider bounding the number of iterations to `MAX_ROYALTY_RECIPIENTS_INDEX` as this is already enforced by `_distributeFunds()`. It may be useful to identify other areas where the `try` statement will not handle reverts on internal calls.

