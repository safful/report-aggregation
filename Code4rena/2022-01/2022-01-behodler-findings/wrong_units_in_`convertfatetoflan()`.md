## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- resolved
- sponsor confirmed

# [Wrong units in `convertFateToFlan()`](https://github.com/code-423n4/2022-01-behodler-findings/issues/188) 

# Handle

sirhashalot


# Vulnerability details

## Impact

The `convertFateToFlan()` function in LimboDAO.sol appears to perform a calculate with improper units. The variable "flan" should hold a quantity of flan, but the units used in the calculation of flan don't match this. This incorrect calculation can allow users to call this function and receive much more flan than the fateToFlan exchange rate specifies.

## Proof of Concept

The `convertFateToFlan()` function is in [the LimboDAO.sol contract](https://github.com/code-423n4/2022-01-behodler/blob/cedb81273f6daf2ee39ec765eef5ba74f21b2c6e/contracts/DAO/LimboDAO.sol#L239)
```
  function convertFateToFlan(uint256 fate) public returns (uint256 flan) {
    require(fateToFlan > 0, "LimboDAO: Fate conversion to Flan disabled.");
    fateState[msg.sender].fateBalance -= fate;
    flan = (fateToFlan * fate) / ONE;
    Flan(domainConfig.flan).mint(msg.sender, flan);
  }
```

The line where flan is calculated multiplies fateToFlan and fate. The units of fateToFlan are "fate / flan" while the units of fate are "fate". The product of the two has units "fate^2 / flan" as shown below:
```
fate     fate      fate ^ 2
——   x         =   ————
flan                flan
```

## Recommended Mitigation Steps

I see two solutions:
1. Modify the line calculating flan to `flan = fate / (fateToFlan * ONE);`
2. Either the fateToFlan value should be renamed to "flanToFate". This would require renaming the `setFateToFlan()` function, the comments describing the respective variable and function, and the unit tests

