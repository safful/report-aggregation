## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Auction collateralToken won't work if token is fee-on-transfer token ](https://github.com/code-423n4/2021-11-malt-findings/issues/227) 

# Handle

harleythedog


# Vulnerability details

## Impact
There are several ERC20 tokens that take a small fee on transfers/transferFroms (known as "fee-on-transfer" tokens). Most notably, USDT is an ERC20 token that has togglable transfer fees, but for now the fee is set to 0 (see the contract here: https://etherscan.io/address/0xdAC17F958D2ee523a2206206994597C13D831ec7#code). For these tokens, it should not be assumed that if you transfer `x` tokens to an address, that the address actually receives `x` tokens. In the current test environment, DAI is the only `collateralToken` available, so there are no issues. However, it has been noted that more pools will be added in the future, so special care will need to be taken if fee-on-transfer tokens (like USDT) are planned to be used as `collateralTokens`.

For example, consider the function `purchaseArbitrageTokens` in Auction.sol. This function transfers `realCommitment` amount of `collateralToken` to the liquidityExtension, and then calls `purchaseAndBurn(realCommitment)` on the liquidityExtension. The very first line of `purchaseAndBurn(amount)` is `require(collateralToken.balanceOf(address(this)) >= amount, "Insufficient balance");`. In the case of fee-on-transfer tokens, this line will revert due to the small fee taken. This means that all calls to `purchaseArbitrageTokens` will fail, which would be very bad when the price goes below peg, since no one would be able to participate in this auction.

## Proof of Concept
See `purchaseArbitrageTokens` here: https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L177

See `purchaseAndBurn` here: https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/LiquidityExtension.sol#L117

## Tools Used
Inspection

## Recommended Mitigation Steps
Add logic to transfers/transferFroms to calculate exactly how many tokens were actually sent to a specific address. In the example given with `purchaseArbitrageTokens`, instead of calling `purchaseAndBurn` with `realCommitment`, the contract should use the difference in the liquidityExtension balance after the transfer minus the liquidityExtension  balance before the transfer.

