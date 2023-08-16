## Tags

- bug
- disagree with severity
- sponsor confirmed
- 3 (High Risk)

# [`LendingPair.liquidateAccount` fails if tokens are lent out](https://github.com/code-423n4/2021-07-wildcredit-findings/issues/123) 

# Handle

cmichel


# Vulnerability details

The `LendingPair.liquidateAccount` function tries to pay out underlying supply tokens to the liquidator using `_safeTransfer(IERC20(supplyToken), msg.sender, supplyOutput)` but there's no reason why there should be enough `supplyOutput` amount in the contract, the contract only ensures `minReserve`.

## Impact
No liquidations can be performed if all tokens are lent out.
Example: User A supplies 1k$ WETH, User B supplies 1.5k$ DAI and borrows the ~1k$ WETH (only leaves `minReserve`). The ETH price drops but user B cannot be liquidated as there's not enough WETH in the pool anymore to pay out the liquidator.

## Recommendation
Mint LP supply tokens to `msg.sender` instead, these are the LP supply tokens that were burnt from the borrower. This way the liquidator basically seizes the borrower's LP tokens.

