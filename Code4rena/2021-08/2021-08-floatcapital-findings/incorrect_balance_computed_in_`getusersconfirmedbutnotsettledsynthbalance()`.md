## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- disagree with severity

# [Incorrect balance computed in `getUsersConfirmedButNotSettledSynthBalance()`](https://github.com/code-423n4/2021-08-floatcapital-findings/issues/142) 

# Handle

hack3r-0m


# Vulnerability details

Consider the following state:

long_synth_balace = 300;
short_synth_balace = 200;

marketUpdateIndex[1] = x;
userNextPrice_currentUpdateIndex = 0;
userNextPrice_syntheticToken_toShiftAwayFrom_marketSide[1][true] = 0;
batched_amountSyntheticToken_toShiftAwayFrom_marketSide[1][true] = 0;

User calls shiftPositionFromLongNextPrice(marketIndex=1, amountSyntheticTokensToShift=100)

This results in following state changes:

long_synth_balace = 200;
short_synth_balace = 200;
userNextPrice_syntheticToken_toShiftAwayFrom_marketSide[1][true] = 100;
batched_amountSyntheticToken_toShiftAwayFrom_marketSide[1][true] = 100;
userNextPrice_currentUpdateIndex = x+1 ;


Due to some other transactions, oracle updates twice, and now the marketUpdateIndex[1] is x+2 and also updating price snapshots.

When User calls getUsersConfirmedButNotSettledSynthBalance(user, 1)

initial condition
```
if (
      userNextPrice_currentUpdateIndex[marketIndex][user] != 0 &&
      userNextPrice_currentUpdateIndex[marketIndex][user] <= currentMarketUpdateIndex
    ) 
```
will be true;

syntheticToken_priceSnapshot[marketIndex][isLong][currentMarketUpdateIndex]
(https://github.com/hack3r-0m/2021-08-floatcapital/blob/main/contracts/contracts/LongShort.sol#L532)

this uses price of current x+2 th update while it should balance of accounting for price of x+1 th update.

