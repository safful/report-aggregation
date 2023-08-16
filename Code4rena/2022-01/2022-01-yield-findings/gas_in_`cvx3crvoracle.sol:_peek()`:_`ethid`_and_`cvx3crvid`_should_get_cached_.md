## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas in `Cvx3CrvOracle.sol:_peek()`: `ethId` and `cvx3CrvId` should get cached ](https://github.com/code-423n4/2022-01-yield-findings/issues/70) 

# Handle

Dravee


# Vulnerability details

## Impact
SLOADs are expensive (~100 gas) compared to MLOADs/MSTOREs (~3 gas). Minimizing them can save gas.

## Proof of Concept
The code is as such (see `@audit-info`):
```
File: Cvx3CrvOracle.sol
110:     function _peek(
111:         bytes6 base,
112:         bytes6 quote,
113:         uint256 baseAmount
114:     ) private view returns (uint256 quoteAmount, uint256 updateTime) {
115:         require(
116:             (base == ethId && quote == cvx3CrvId) || // @audit-info ethId SLOAD 1, cvx3CrvId SLOAD 1
117:                 (base == cvx3CrvId && quote == ethId), // @audit-info ethId SLOAD 2, cvx3CrvId SLOAD 2
118:             "Invalid quote or base"
119:         );
120:         (, int256 daiPrice, , , ) = DAI.latestRoundData();
121:         (, int256 usdcPrice, , , ) = USDC.latestRoundData();
122:         (, int256 usdtPrice, , , ) = USDT.latestRoundData();
123: 
124:         require(
125:             daiPrice > 0 && usdcPrice > 0 && usdtPrice > 0,
126:             "Chainlink pricefeed reporting 0"
127:         );
128: 
129:         // This won't overflow as the max value for int256 is less than the max value for uint256
130:         uint256 minStable = min(
131:             uint256(daiPrice),
132:             min(uint256(usdcPrice), uint256(usdtPrice))
133:         );
134: 
135:         uint256 price = (threecrv.get_virtual_price() * minStable) / 1e18;
136: 
137:         if (base == cvx3CrvId && quote == ethId) { // @audit-info ethId SLOAD 3, cvx3CrvId SLOAD 3
138:             quoteAmount = (baseAmount * price) / 1e18;
139:         } else {
140:             quoteAmount = (baseAmount * 1e18) / price;
141:         }
142: 
143:         updateTime = block.timestamp;
144:     }
```

By caching `ethId` and `cvx3CrvId` in memory, it's possible to save 4 SLOADs (~400gas) at the cost of 2 MSTOREs (6 gas) and 4 MLOADs (12 gas)

## Tools Used
VS Code

## Recommended Mitigation Steps
Cache `ethId` and `cvx3CrvId` in variables and use these instead

