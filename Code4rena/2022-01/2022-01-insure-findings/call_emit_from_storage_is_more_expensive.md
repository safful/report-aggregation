## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [call emit from storage is more expensive](https://github.com/code-423n4/2022-01-insure-findings/issues/124) 

# Handle

Fitraldys


# Vulnerability details

## Impact
in line https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L685 the function emitted a `MarketStatusChanged` event with storage variable which is `marketStatus`.
when we emit an event using storage data is more expensive than emitted an event using `MarketStatus.Payingout` value. 

## Proof of Concept
https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L685
```
contract emitstatust {

    enum MarketStatus {
        Trading,
        Payingout
    }
    MarketStatus public marketStatus;

    event MarketStatusChanged(MarketStatus statusValue);

    function amit() public {
    
    marketStatus = MarketStatus.Payingout;

    emit MarketStatusChanged(marketStatus);

    }
}
//44792 gas
```

can change to :

```
contract emitstatust {

    enum MarketStatus {
        Trading,
        Payingout
    }
    MarketStatus public marketStatus;

    event MarketStatusChanged(MarketStatus statusValue);

    function amit() public {
    
    marketStatus = MarketStatus.Payingout;

    emit MarketStatusChanged(MarketStatus.Payingout);

    }
}
//44659 gas
```

## Tools Used
remix


