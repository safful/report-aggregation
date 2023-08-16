## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- resolved
- fixed-in-upstream-repo

# [latestMarket used where marketIndex should have been used](https://github.com/code-423n4/2021-08-floatcapital-findings/issues/9) 

# Handle

gpersoon


# Vulnerability details

## Impact
The functions initializeMarket and _seedMarketInitially use the variable latestMarket.
If these functions would be called seperately from createNewSyntheticMarket, then latestMarket would have the same value for each call of initializeMarket and _seedMarketInitially 

This would mean that the latestMarket is initialized multiple times and the previous market(s) are not initialized properly.
Note: the call to addNewStakingFund could have prevented this issue, but also allows this, see separate issue.

Note: the functions can only be called by the admin, so if createNewSyntheticMarket and initializeMarket are called in combination, then it would not lead to problems,
but in future release of the software the calls to createNewSyntheticMarket and initializeMarket might get separated.

## Proof of Concept
//https://github.com/code-423n4/2021-08-floatcapital/blob/main/contracts/contracts/LongShort.sol#L304
function _seedMarketInitially(uint256 initialMarketSeedForEachMarketSide, uint32 marketIndex) internal virtual {
   ...
    ISyntheticToken(syntheticTokens[latestMarket][true]).mint(PERMANENT_INITIAL_LIQUIDITY_HOLDER,initialMarketSeedForEachMarketSide);   // should be marketIndex
    ISyntheticToken(syntheticTokens[latestMarket][false]).mint(PERMANENT_INITIAL_LIQUIDITY_HOLDER,initialMarketSeedForEachMarketSide);  // should be marketIndex

  function initializeMarket(
     uint32 marketIndex,....)
  ...
    require(!marketExists[marketIndex], "already initialized");
    require(marketIndex <= latestMarket, "index too high"); 
    marketExists[marketIndex] = true;
..
    IStaker(staker).addNewStakingFund(
      latestMarket,                                       // should be marketIndex
      syntheticTokens[latestMarket][true],   // should be marketIndex
      syntheticTokens[latestMarket][false],  // should be marketIndex
  ...

## Tools Used

## Recommended Mitigation Steps
Replace latestMarket with marketIndex in the functions initializeMarket and _seedMarketInitially

p.s. confirmed by Jason of float capital: Definitely an issue, luckily both of those functions are adminOnly. But that is definitely not ideal!

