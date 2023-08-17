## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed
- reviewed

# [Position owner should set allowed slippage](https://github.com/code-423n4/2022-04-backd-findings/issues/87) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/actions/topup/TopUpAction.sol#L154
https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/actions/topup/TopUpAction.sol#L187


# Vulnerability details

## Impact
The default swap slippage of 5% allows malicious keepers to sandwich attack topup. Additionally, up to 40% (_MIN_SWAPPER_SLIPPAGE) slippage allows malicious owner to sandwich huge amounts from topup

## Proof of Concept
Keeper can bundle swaps before and after topup to sandwich topup action, in fact it's actually in their best interest to do so.

## Tools Used

## Recommended Mitigation Steps
Allow user to specify max swap slippage when creating topup similar to how it's specified on uniswap or sushiswap to block attacks from both keepers and owners

