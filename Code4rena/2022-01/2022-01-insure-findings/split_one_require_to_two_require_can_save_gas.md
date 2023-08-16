## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [split one require to two require can save gas](https://github.com/code-423n4/2022-01-insure-findings/issues/113) 

# Handle

Fitraldys


# Vulnerability details

## Impact
in line https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L260 have two check inside the require which is `marketStatus == MarketStatus.Trading` and `paused == false` and by spliting this check we can save gas. 

## Proof of Concept
https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L260

```
function woi() public {

        require(
            marketStatus == MarketStatus.Trading && paused == false,
            "ERROR: DEPOSIT_DISABLED"
        );

    }
// 23645 gas
``` 

can be change to 

```
function woi() public{ 

        require(
            marketStatus == MarketStatus.Trading, "ERROR: DEPOSIT_DISABLED"
        );
        require(  
            paused == false, "ERROR: DEPOSIT_DISABLED"
        );

    }
//23637 gas
```




