## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`treasuryShare` is Overwritten in `FeePoolV0._shareMochi()`](https://github.com/code-423n4/2021-10-mochi-findings/issues/89) 

# Handle

leastwood


# Vulnerability details

## Impact

The `FeePoolV0.sol` contract accrues fees upon the liquidation of undercollaterised positions. These fees are split between treasury and `vMochi` contracts. However, when `distributeMochi()` is called to distribute `mochi` tokens to `veCRV` holders, both `mochiShare` and `treasuryShare` is flushed from the contract when there are still `usdm` tokens in the contract. 

## Proof of Concept

Consider the following scenario:
  - The `FeePoolV0.sol` contract contains 100 `usdm` tokens at an exchange rate of 1:1 with `mochi` tokens.
  - `updateReserve()` is called to set the split of `usdm` tokens such that `treasuryShare` has claim on 20 `usdm` tokens and `mochiShare` has claim on the other 80 tokens.
  - A `veCRV` holder seeks to increase their earnings by calling `distributeMochi()` before `sendToTreasury()` has been called.
  - As a result, 80 `usdm` tokens are converted to `mochi` tokens and  locked in a curve rewards pool.
  - Consequently, `mochiShare` and `treasuryShare` is set to `0` (aka flushed).
  - The same user calls `updateReserve()` to split the leftover 20 `usdm` tokens between `treasuryShare` and `mochiShare`. 
  - `mochiShare` is now set to 16 `usdm` tokens.
  - The above process is repeated to distribute `mochi` tokens to `veCRV` holders again and again.
  - The end result is that `veCRV` holders have been able to receive all tokens that were intended to be distributed to the treasury.

https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/feePool/FeePoolV0.sol#L94

## Tools Used

Manual code review
Discussions with the Mochi team.

## Recommended Mitigation Steps

Consider removing the line in `FeePoolV0.sol` (mentioned above), where `treasuryShare` is flushed. 

