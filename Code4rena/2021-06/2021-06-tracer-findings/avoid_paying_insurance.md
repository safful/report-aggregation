## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- disagree with severity

# [avoid paying insurance](https://github.com/code-423n4/2021-06-tracer-findings/issues/30) 

# Handle

gpersoon


# Vulnerability details

## Impact
It's possible to avoid paying insurance in the following way:
- once per hour (at the right moment), do the following:
----using a flash loan, or with a large amount of tokens, call deposit of Insurance.sol to make sure that the pool is sufficiently filled (poolHoldings > poolTarget)
----call the function executeTrade of Trader.sol with a minimal trade (possibly of value 0, see finding "executeTrade with same trades")
----executeTrade calls matchOrders, which calls recordTrade
----recordTrade calls updateFundingRate();   (once per hour, so you have to be sure you do it in time before other trades trigger this)
----updateFundingRate calls getPoolFundingRate
----getPoolFundingRate determines the insurance rate, but because the insurance pool is sufficiently full (due to the flash loan), the rate is 0
----updateFundingRate stores the 0 rate via setInsuranceFundingRate  (which is used later on to calculate the amounts for the insurances)
----withdraw from the Insurance and pay back the flash loan

The insurance rates are 0 now and no-one pays insurance.
The gas costs relative to the insurance costs + the flash loan fees determine if this is an economically viable attack. Otherwise it is still a grief attack
This will probably be detected pretty soon because the insurance pool will stay empty. However its difficult to prevent.

## Proof of Concept

// https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Insurance.sol#L45
function deposit(uint256 amount) external override {

// https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Insurance.sol#L74
function withdraw(uint256 amount) external override {

// https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Pricing.sol#L69     
function recordTrade(uint256 tradePrice) external override onlyTracer {
      ..
        if (startLastHour <= block.timestamp - 1 hours) {
           ..
            updateFundingRate();

// https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Pricing.sol#L141
function updateFundingRate() internal {
      ..
        int256 iPoolFundingRate = insurance.getPoolFundingRate().toInt256();
      ..
        int256 iPoolFundingRateValue = currentInsuranceFundingRateValue + iPoolFundingRate;
     ..
        setInsuranceFundingRate(iPoolFundingRate, iPoolFundingRateValue);

// https://github.com/code-423n4/2021-06-tracer/blob/main/src/contracts/Insurance.sol#L204
function getPoolFundingRate() external view override returns (uint256) {
     ..
        // If the pool is above the target, we don't pay the insurance funding rate
        if (poolTarget <= poolHoldings) {
            return 0;
        }

## Tools Used

## Recommended Mitigation Steps
Set a timelock on withdrawing insurance


