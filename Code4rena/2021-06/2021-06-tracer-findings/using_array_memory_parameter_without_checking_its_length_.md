## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Using array memory parameter without checking its length ](https://github.com/code-423n4/2021-06-tracer-findings/issues/35) 

# Handle

JMukesh


# Vulnerability details

## Impact
These array memory parameter can be  problematic if not used properly , if the array is very large it may overlap over other part of memory.

## Proof of Concept

https://github.com/code-423n4/2021-06-tracer/blob/74e720ee100fd027c592ea44f272231ad4dfa2ab/src/contracts/Liquidation.sol#L274

This an example to show the exploit:
// based on https://github.com/paradigm-operations/paradigm-ctf-2021/blob/master/swap/private/Exploit.sol
pragma solidity ^0.4.24; // only works with low solidity version

contract test{
struct Overlap {
uint field0;
}
event log(uint);

function mint(uint[] memory amounts) public returns (uint) { 
// this can be in any solidity version
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

manual review

## Recommended Mitigation Steps

check array length before using it

