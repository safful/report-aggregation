## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unnecessary new list in Basket's validateWeights()](https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/56) 

# Handle

kenzo


# Vulnerability details

# Vulnerability details

The function creates and populates a new array to check for duplicates, this is not necessary.

## Impact
Some amount of gas unnecessarily spent.

## Proof of Concept
The relevant area:
https://github.com/code-423n4/2021-10-defiprotocol/blob/main/contracts/contracts/Basket.sol#L71:#L80
## Tools Used
Manual analysis, hardhat gas estimator.

## Recommended Mitigation Steps
Change the check to the following:
```
for (uint i = 0; i < length; i++) {
  require(_tokens[i] != address(0));
  require(_weights[i] > 0);
  for (uint256 x = 0; x < i; x++) {
      require(_tokens[i] != _tokens[x]);
  }
}
```

