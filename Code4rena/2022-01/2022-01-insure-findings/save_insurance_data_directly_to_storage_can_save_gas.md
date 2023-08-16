## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [save Insurance data directly to storage can save gas](https://github.com/code-423n4/2022-01-insure-findings/issues/122) 

# Handle

Fitraldys


# Vulnerability details

## Impact
in line https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L508 instead of save `Insurance` value to memory then save to `insurences` storage it's better to save the `Insurence` value directly to `insurences`  storage or mapping to save gas.

## Proof of Concept
https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L508
```
contract insur {

    struct Insurance {
        uint256 id; //each insuance has their own id
        uint256 startTime; //timestamp of starttime
        uint256 endTime; //timestamp of endtime
        uint256 amount; //insured amount
        bytes32 target; //target id in bytes32
        address insured; //the address holds the right to get insured
        bool status; //true if insurance is not expired or redeemed
    }

     mapping(uint256 => Insurance) public insurances;

    function coba() public {

        uint256 _id = 10;
        uint256 _endTime = 10;
        uint256 _amount = 12;
        bytes32 _target = bytes32(uint256(10));


        Insurance memory _insurance = Insurance(
            _id,
            block.timestamp,
            _endTime,
            _amount,
            _target,
            msg.sender,
            true
        );
        insurances[_id] = _insurance;

    }
}
//154623 gas
```
 change to :
```
contract insur {

    struct Insurance {
        uint256 id; //each insuance has their own id
        uint256 startTime; //timestamp of starttime
        uint256 endTime; //timestamp of endtime
        uint256 amount; //insured amount
        bytes32 target; //target id in bytes32
        address insured; //the address holds the right to get insured
        bool status; //true if insurance is not expired or redeemed
    }

     mapping(uint256 => Insurance) public insurances;

    function coba() public {

        uint256 _id = 10;
        uint256 _endTime = 10;
        uint256 _amount = 12;
        bytes32 _target = bytes32(uint256(10));


        insurances[_id] = Insurance(
            _id,
            block.timestamp,
            _endTime,
            _amount,
            _target,
            msg.sender,
            true
        );
        

    }
}
//154610 gas 
```

## Tools Used
remix

