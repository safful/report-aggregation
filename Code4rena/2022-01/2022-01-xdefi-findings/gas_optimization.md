## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [gas optimization](https://github.com/code-423n4/2022-01-xdefi-findings/issues/103) 

# Handle

Fitraldys


# Vulnerability details

## Impact
expensive gas, because in the line https://github.com/XDeFi-tech/xdefi-distribution/blob/v1.0.0-beta.0/contracts/XDEFIDistributionHelper.sol#L23, 
the tokenids.length is save to a new variable to be used in the for loop, instead of call tokenids.length directly in the for loop

## Proof of Concept
```
pragma solidity =0.8.7;

contract pikir {

    function putar1 (uint256 [] memory tokenIds) external view returns(uint256) {
        
        uint256 alltokens = tokenIds.length;
        uint256 hasil;

        for (uint256 i; i < alltokens; ++i){
            
            hasil += 1;

        }
        return hasil;

    }

}
//24714 gas

contract pikir2 {

    function putar1 (uint256 [] memory tokenIds) external view returns(uint256) {
    
        uint256 hasil;

        for (uint256 i; i < tokenIds.length; ++i){
            
            hasil += 1;

        }
        return hasil;

    }

}
//24710 gas
```

## Tools Used
remix


