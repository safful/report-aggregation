## Tags

- bug
- duplicate
- G (Gas Optimization)
- sponsor confirmed
- resolved

# [Gas optimization for withdraw and withdrawAll](https://github.com/code-423n4/2021-08-floatcapital-findings/issues/44) 

# Handle

0xImpostor


# Vulnerability details

## Impact

Use `marketIndex` to updateSystemState instead of querying `marketIndexOfToken[token]` twice.

## Proof of Concept

Swap line 949 with 951 and swap line 974 with 976.

For example

```
function withdrawAll(address token) external {
    uint32 marketIndex = marketIndexOfToken[token];

    ILongShort(longShort).updateSystemState(marketIndex);

    uint256 amountToShiftForThisToken = syntheticTokens[marketIndex][true] == token
      ? userNextPrice_amountStakedSyntheticToken_toShiftAwayFrom_long[marketIndex][msg.sender]
      : userNextPrice_amountStakedSyntheticToken_toShiftAwayFrom_short[marketIndex][msg.sender];

    _withdraw(marketIndex, token, userAmountStaked[token][msg.sender] - amountToShiftForThisToken);
  }

```

## Tools Used

Manual analysis

