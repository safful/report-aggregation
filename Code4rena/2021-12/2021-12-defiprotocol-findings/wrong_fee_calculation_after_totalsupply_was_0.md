## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Wrong fee calculation after totalSupply was 0](https://github.com/code-423n4/2021-12-defiprotocol-findings/issues/58) 

# Handle

kenzo


# Vulnerability details

`handleFees` does not update `lastFee` if `startSupply == 0`.
This means that wrongly, extra fee tokens would be minted once the basket is resupplied and `handleFees` is called again.

## Impact
Loss of user funds.
The extra minting of fee tokens comes on the expense of the regular basket token owners, which upon withdrawal would get less underlying than their true share, due to the dilution of their tokens' value.

## Proof of Concept
Scenario:
- All basket token holders are burning their tokens. The last burn would set totalSupply to 0.
- After 1 day, somebody mints basket tokens.
`handleFees` would be called upon mint, and would just return since totalSupply == 0. Note: It does not update `lastFee`.
```
} else if (startSupply == 0) {
            return;
```
https://github.com/code-423n4/2021-12-defiprotocol/blob/main/contracts/contracts/Basket.sol#L136:#L137
- The next block, somebody else mints a token. Now `handleFees` will be called and will calculate the fees according to the current supply and the time diff between now and `lastFee`:
```
uint256 timeDiff = (block.timestamp - lastFee);
```
https://github.com/code-423n4/2021-12-defiprotocol/blob/main/contracts/contracts/Basket.sol#L139
But as we saw, `lastFee` wasn't updated in the previous step. `lastFee` is still the time of 1 day before - when the last person burned his tokens and the basket supply was 0.
So now the basket will mint fees as if a whole day has passed since the last calculation, but actually it only needs to calculate the fees for the last block, since only then we had tokens in the basket.

## Recommended Mitigation Steps
Set `lastFee = block.timestamp` if `startSupply == 0`.

