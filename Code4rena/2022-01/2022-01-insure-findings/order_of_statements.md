## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Order of statements](https://github.com/code-423n4/2022-01-insure-findings/issues/332) 

# Handle

pauliax


# Vulnerability details

## Impact
Statements should be ordered in a way that it costs less gas, that is, less operations are performed when the validating conditions are wrong.
e.g. this can be reordered:
```solidity
  //Distribute premium and fee
  uint256 _endTime = _span + block.timestamp;
  uint256 _premium = getPremium(_amount, _span);
  uint256 _fee = parameters.getFeeRate(msg.sender);

  require(
      _amount <= availableBalance(),
      "ERROR: INSURE_EXCEEDED_AVAILABLE_BALANCE"
  );
  require(_premium <= _maxCost, "ERROR: INSURE_EXCEEDED_MAX_COST");
  require(_span <= 365 days, "ERROR: INSURE_EXCEEDED_MAX_SPAN");
  require(
      parameters.getMinDate(msg.sender) <= _span,
      "ERROR: INSURE_SPAN_BELOW_MIN"
  );

  require(
      marketStatus == MarketStatus.Trading,
      "ERROR: INSURE_MARKET_PENDING"
  );
  require(paused == false, "ERROR: INSURE_MARKET_PAUSED");
```
to something like this:
```solidity
  require(paused == false, "ERROR: INSURE_MARKET_PAUSED");
  require(
      marketStatus == MarketStatus.Trading,
      "ERROR: INSURE_MARKET_PENDING"
  );

  require(
      _amount <= availableBalance(),
      "ERROR: INSURE_EXCEEDED_AVAILABLE_BALANCE"
  );

  require(_span <= 365 days, "ERROR: INSURE_EXCEEDED_MAX_SPAN");
  require(
      parameters.getMinDate(msg.sender) <= _span,
      "ERROR: INSURE_SPAN_BELOW_MIN"
  );

  //Distribute premium and fee
  uint256 _premium = getPremium(_amount, _span);
  require(_premium <= _maxCost, "ERROR: INSURE_EXCEEDED_MAX_COST");

  uint256 _endTime = _span + block.timestamp;
  uint256 _fee = parameters.getFeeRate(msg.sender);
```

