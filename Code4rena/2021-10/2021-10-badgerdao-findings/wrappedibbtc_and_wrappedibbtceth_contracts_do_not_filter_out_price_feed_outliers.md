## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [WrappedIbbtc and WrappedIbbtcEth contracts do not filter out price feed outliers](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/87) 

# Handle

hyh


# Vulnerability details

## Impact
If price feed is manipulated in any way or there is any malfunction based volatility on the market, both contracts will pass it on a user.
In the same time it's possible to construct mitigation mechanics for such cases, so user economics be affected by sustainable price movements only.
As price outrages provide a substantial attack surface for the project it's worth adding some complexity to the implementation.

## Proof of Concept
In WrappedIbbtcEth pricePerShare variable is updated by externally run updatePricePerShare function, https://github.com/code-423n4/2021-10-badgerdao/blob/main/contracts/WrappedIbbtcEth.sol#L72, and then used in mint/burn/transfer functions without additional checks via balanceToShares function: https://github.com/code-423n4/2021-10-badgerdao/blob/main/contracts/WrappedIbbtcEth.sol#L155.

In WrappedIbbtc price is requested via pricePerShare function, https://github.com/code-423n4/2021-10-badgerdao/blob/main/contracts/WrappedIbbtc.sol#L123, and used in the same way without additional checks via balanceToShares function, https://github.com/code-423n4/2021-10-badgerdao/blob/main/contracts/WrappedIbbtc.sol#L147. 

## Recommended Mitigation Steps
Introduce minting/burning query that runs on schedule, separating user funds contribution and actual mint/burn. With user deposit or burn the corresponding action to be added to commitment query, which execution for mint or redeem will later be sparked by off-chain script according to fixed schedule.
This also can be open to public execution with gas compensation incentive, for example as it's done in Tracer protocol: https://github.com/tracer-protocol/perpetual-pools-contracts/blob/develop/contracts/implementation/PoolKeeper.sol#L131
Full code of an implementation is too big to include it in the report, but viable versions are available publicly (Tracer protocol version can be found at the same repo, https://github.com/tracer-protocol/perpetual-pools-contracts/blob/develop/contracts/implementation/PoolCommitter.sol).

Once the scheduled mint/redeem query is added, the additional logic to control for price outliers will become possible there, as in this case mint/redeem execution can be conditioned to happen on calm market only, where various definitions of calm can be implemented.
One of the approaches is to keep track of recent prices and require that new price each time be within a threshold from median of their array.

Example:

// Introduce small price tracking arrays:
uint256[] private times;
uint256[] private prices;

// Current position in array
uint8 curPos;

// Current length, grows from 0 to totalMaxPos as prices are being added
uint8 curMaxPos;

// Maximum length, we track up to totalMaxPos prices
uint8 totalMaxPos = 10;

// Price movement threshold
uint256 moveThreshold = 0.1*1e18;

We omit the full implementation here as it is lengthy enough and can vary.
The key steps are:
* Run query for scheduled mint/redeem with logic: if next price is greater than median of currently recorded prices by threshold, add it to the records, but do not mint/redeem.
* That is, when scheduled mint/redeem is run, on new price request, WrappedIbbtcEth.core.pricePerShare() or WrappedIbbtc.oracle.pricePerShare(), get newPrice and calculate current price array median, curMed
* prices[curPos] = newPrice
* if (curMaxPos < totalMaxPos) {curMaxPos += 1}
* if (curPos == curMaxPos) {curPos = 0} else {curPos += 1}
* if (absolute_value_of(newPrice - curMed) < moveThreshold * curMed / 1e18) {do_mint/redeem; return_0_status}
* else {return_1_status}

Schedule should be frequent enough, say once per 30 minutes, which is kept while returned status is 0. While threshold condition isn't met and returned status is 1, it runs once per 10 minutes. The parameters here are subject to calibration.
This way if the price movement is sustained the mint/redeem happens after price array median comes to a new equilibrium. If price reverts, the outbreak will not have material effect mint/burn operations. This way the contract vulnerability is considerably reduced as attacker would need to keep distorted price for period long enough, which will happen after the first part of deposit/withdraw cycle. I.e. deposit and mint, burn and redeem operations will happen not simultaneously, preventing flash loans to be used to elevate the quantities, and for price to be effectively distorted it would be needed to keep it so for substantial amount of time.

