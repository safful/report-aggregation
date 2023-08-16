## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Malt Protocol Uses Stale Results From `MaltDataLab` Which Can Be Abused By Users](https://github.com/code-423n4/2021-11-malt-findings/issues/373) 

# Handle

leastwood


# Vulnerability details

## Impact

`MaltDataLab` integrates several `MovingAverage` contracts to fetch sensitive data for the Malt protocol. Primary data used by the protocol consists of the real value for LP tokens, the average price for Malt and average reserve ratios. `trackMaltPrice`, `trackPoolReserves` and `trackPool` are called by a restricted role denoted as the `UPDATER_ROLE` and represented by an EOA account and not another contract. Hence, the EOA account must consistently update the aforementioned functions to ensure the most up-to-date values. However, miners can censor calls to `MaltDataLab` and effectively extract value from other areas of the protocol which use stale values.

## Proof of Concept

Consider the following attack vector:
- The price of Malt exceeds the lower bound threshold and hence `stabilize` can be called by any user.
- The `_stabilityWindowOverride` function is satisfied, hence the function will execute.
- The state variable, `exchangeRate`, queries `maltPriceAverage` which may use an outdated exchange rate.
- `_startAuction` is executed which rewards `msg.sender` with 100 Malt as an incentive for triggering an auction.
- As the price is not subsequently updated, a malicious attacker could collude with a miner to censor further pool updates and continue calling `stabilize` on every `fastAveragePeriod` interval to extract incentive payments.
- If the payments exceed what the `UPDATER_ROLE` is willing to pay to call `trackMaltPrice`, a user is able to sustain this attack.

This threatens the overall stability of the protocol and should be properly handled to prevent such attacks. However, the fact that `MaltDataLab` uses a series of spot price data points to calculate the `MovingAverage` also creates an area of concern as well-funded actors could still manipulate the `MovingAverage` contract by sandwiching calls to `trackMaltPrice`, `trackPool` and `trackPoolReserves`.

`trackMaltPrice`, `trackPool`, and `trackPoolReserves` should be added to the following areas of the code where applicable.
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Bonding.sol#L159
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Bonding.sol#L173
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Bonding.sol#L177
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Auction.sol#L881
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Auction.sol#L710
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/StabilizerNode.sol#L156
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/StabilizerNode.sol#L190
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/ImpliedCollateralService.sol#L105

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider adding calls to `trackMaltPrice`, `trackPoolReserves` and `trackPool` wherever the values are impacted by the protocol. This should ensure the protocol is tracking the most up-to-date values. Assuming the cumulative values are used in the `MovingAverage` contracts, then sensitive calls utilising `MaltDataLab` should be protected from flashloan attacks. However, currently this is not the case, rather `MovingAverage` consists of a series of spot price data points which can be manipulated by well-funded actors or via a flashloan. Therefore, there needs to be necessary changes made to `MaltDataLab` to use cumulative price updates as its moving average instead of spot price.

