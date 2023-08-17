## Tags

- bug
- 3 (High Risk)
- disagree with severity
- sponsor confirmed

# [Total Supply is not guaranteed and is not deterministic.](https://github.com/code-423n4/2022-05-backd-findings/issues/46) 

# Lines of code

 https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/tokenomics/Minter.sol#L181


# Vulnerability details

## Impact
The actual total supply of the token is random and depends on when `_executeInflationRateUpdate` is executed.

## Proof of concept
The `README` and tokenomic documentation clearly states that “The token supply is limited to a total of 268435456 tokens.”. However when executing [`_executeInflationRateUpdate`](https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/tokenomics/Minter.sol#L181), it first uses the current inflation rate to update the total available before checking if it needs to be reduced. 

Therefore if no one mints or calls `executeInflationRateUpdate` for some time around the decay point, the inflation will be updated using the previous rate so the  `totalAvailableToNow` will grow too much.

## Mitigation steps

You should do
```js 
totalAvailableToNow += (currentTotalInflation * (block.timestamp - lastEvent));
```

Only if the condition `block.timestamp >= lastInflationDecay + _INFLATION_DECAY_PERIOD` is false. 

Otherwise you should do 

```js 
totalAvailableToNow += (currentTotalInflation * (lastInflationDecay + _INFLATION_DECAY_PERIOD - lastEvent)); 
```

Then update the rates, then complete with
```js 
totalAvailableToNow += (currentTotalInflation * (block.timestamp - lastInflationDecay + _INFLATION_DECAY_PERIOD)); 
```

Note that as all these variables are either constants either already loaded in memory this is super cheap to do.


