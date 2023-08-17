## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Swivel lend method doesn't pull protocol fee from user](https://github.com/code-423n4/2022-06-illuminate-findings/issues/201) 

# Lines of code

https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L297


# Vulnerability details

The Swivel `lend` method adds to `fees[u]` the order fee, but does not pull that fee from the user. It only pulls the order-post-fee amount.

## Impact
`withdrawFee` will fail, as it tries to transfer more tokens than are in the contract.

## Proof of Concept
The Swivel `lend` method [sums up](https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L279:#L283) the fees to `totalFee`, and the amount to send to Swivel in `lent`:
```
                    totalFee += fee;
                    // Amount lent for this order
                    uint256 amountLent = amount - fee;
                    // Sum the total amount lent to Swivel (amount of ERC5095 tokens to mint) minus fees
                    lent += amountLent;
```
It then [increments](https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L294:#L297) `fees[u]` by `totalFee`, but only pulls from the user `lent`:
```
            fees[u] += totalFee;
            // transfer underlying tokens from user to illuminate
            Safe.transferFrom(IERC20(u), msg.sender, address(this), lent);
```
Therefore, `totalFee` has not been pulled from the user.
The `fees` variable now includes tokens which are not in the contract, and `withdrawFee` will fail as [it tries to transfer](https://github.com/code-423n4/2022-06-illuminate/blob/main/lender/Lender.sol#L667) `fees[u]`.

## Recommended Mitigation Steps
Pull `lent + totalFee` from the user.

