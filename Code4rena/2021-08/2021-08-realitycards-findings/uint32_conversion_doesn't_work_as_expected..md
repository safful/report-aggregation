## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Resolved

# [uint32 conversion doesn't work as expected.](https://github.com/code-423n4/2021-08-realitycards-findings/issues/28) 

# Handle

gpersoon


# Vulnerability details

## Impact
The uint32 conversion in setWinner of the RCMarket doesn't work as expected.
The first statement: "uint32(block.timestamp)" already first the block.timestamp in a uint32.
If it is larger than type(uint32).max it wraps around and starts with 0 again
The testcode below shows this.

Check for "<= type(uint32).max" in the second statement is useless because _blockTimestamp is always  <= type(uint32).max

## Proof of Concept
// https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCMarket.sol#L507
 function setWinner(uint256 _winningOutcome) internal {
...
            uint256 _blockTimestamp = uint32(block.timestamp);
            require(_blockTimestamp <= type(uint32).max, "Overflow");


//Testcode:
pragma solidity 0.8.7;
contract Convert {
   uint256 public a = uint256( type(uint32).max )+1; // a==4294967296
   uint32  public b = uint32(a); // b==0
   uint256 public c = uint32(a); // c==0
}
   
## Tools Used

## Recommended Mitigation Steps
Do the require first (without a typecast to uint32):

            require( block.timestamp <= type(uint32).max, "Overflow");
            uint256 _blockTimestamp = uint32(block.timestamp);



