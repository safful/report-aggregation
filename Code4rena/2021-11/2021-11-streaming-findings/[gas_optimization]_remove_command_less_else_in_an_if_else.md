## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [[Gas optimization] remove command less else in an if else](https://github.com/code-423n4/2021-11-streaming-findings/issues/137) 

# Handle

Omik


# Vulnerability details

## Impact
In the https://github.com/code-423n4/2021-11-streaming/blob/main/Streaming/src/Locke.sol#L472 the withdraw and stake function there is unnecessary else statement which didnt have any command inside it, this can lead to gas consumption more expensive then using only if statement for isSale check.

## Proof of Concept
pragma solidity ^0.8.0;


contract testing {

    uint public counter;

    function test()public {
        if(true){
            counter += 1;
        }else{

        }
    }//43582 gas

    function test2()public {
        if(true){
            counter += 1;
        }
    }//26449 gas
    
}

