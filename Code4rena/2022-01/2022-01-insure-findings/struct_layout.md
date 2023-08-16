## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Struct layout](https://github.com/code-423n4/2022-01-insure-findings/issues/253) 

# Handle

Jujic


# Vulnerability details

## Impact
Insurance struct in `PoolTemplate .sol` can be optimized to reduce 2 storage slot

## Proof of Concept
https://github.com/code-423n4/2022-01-insure/blob/19d1a7819fe7ce795e6d4814e7ddf8b8e1323df3/contracts/PoolTemplate.sol#L127-L128
```
struct Insurance {
        uint256 id; //each insuance has their own id
        uint256 startTime; //timestamp of starttime
        uint256 endTime; //timestamp of endtime
        uint256 amount; //insured amount
        bytes32 target; //target id in bytes32
        address insured; //the address holds the right to get insured
        bool status; //true if insurance is not expired or redeemed
    }
```
`startTime` and `endTime `store block numbers, and 2^48 is being enough for a very long time.
## Tools Used
https://docs.soliditylang.org/en/v0.8.0/internals/layout_in_storage.html?highlight=Structs#layout-of-state-variables-in-storage


## Recommended Mitigation Steps
The struct can be changed into:
```
struct Insurance {
        uint256 id; //each insuance has their own id
        uint48 startTime; //timestamp of starttime
        uint48 endTime; //timestamp of endtime
        address insured; //the address holds the right to get insured
        uint256 amount; //insured amount
        bytes32 target; //target id in bytes32
        bool status; //true if insurance is not expired or redeemed
    }
```


