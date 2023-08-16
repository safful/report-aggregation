## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Claim liquidation escrow](https://github.com/code-423n4/2021-06-tracer-findings/issues/2) 

# Handle

gpersoon


# Vulnerability details

## Impact
A liquidator can always claim the liquidation escrow in the following way:
- create a second account
- setup a complimentary trade in that second account, which will result in a large slippage when executed
- call executeTrade (which everyone can call), to execute a trade between his own two accounts with a large slippage
- the slippage doesn't hurt because the liquidator owns both accounts
- call claimReceipt with the receiptId of the executed order, within the required period (e.g. 15 minutes)

## Proof of Concept
 
// https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Trader.sol#L67
function executeTrade(Types.SignedLimitOrder[] memory makers, Types.SignedLimitOrder[] memory takers) external override {

https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Liquidation.sol#L394
 function claimReceipt( uint256 receiptId, Perpetuals.Order[] memory orders, address traderContract) external override {

## Tools Used

## Recommended Mitigation Steps
perhaps limit who can call executeTrade 

