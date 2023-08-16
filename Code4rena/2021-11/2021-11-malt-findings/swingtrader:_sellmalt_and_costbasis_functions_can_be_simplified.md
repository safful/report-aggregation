## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [SwingTrader: sellMalt and costBasis functions can be simplified](https://github.com/code-423n4/2021-11-malt-findings/issues/72) 

# Handle

hyh


# Vulnerability details

## Impact

Gas is overspent on function call and calculations

## Proof of Concept

First, save Malt and Collateral tokens decimals difference to storage variable. As neither Malt, nor Collateral token decimals change since initial setup, both can be saved and accessed as a storage variable instead of calling ```decimals()``` function and calculating the difference each time.

Second, now sellMalt calls costBasis, which already retrieved decimals and their difference, but sellMalt ignores those, retrieving them from functions/storage again. This could be unified as discussed below.

sellMalt:
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/SwingTrader.sol#L77
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/SwingTrader.sol#L109

costBasis:
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/SwingTrader.sol#L146

 ## Recommended Mitigation Steps
 
Save both decimals values to contract storage variables and use them instead of ```decimals()``` function. As the calculations use decimals difference it might be enough to save and use the difference only.
In any case saving is preferred to calling as the latter spend gas on call and storage access anyway.
 
Also, return the difference along with decimals from costBasis and use them in sellMalt instead of obtaining afresh. I.e. first reuse ```costBasis``` returned ```decimals``` instead of ```collateralToken.decimals()```, then add ```maltDecimals``` and the difference, whether ```maltDecimals - decimals``` or ```decimals - maltDecimals``` to its output and use in rewards / soldBasis calculations. Function arguments and returned values are memory and are cheaper than another storage access.

Now:
```
(uint256 basis,) = costBasis();
...
uint256 maltDecimals = malt.decimals();
uint256 decimals = collateralToken.decimals();
...
uint256 diff = maltDecimals - decimals; 
```
To be:
```
(uint256 basis, uint256 decimals, uint256 maltDecimals, uint256 diff) = costBasis();
...
```

