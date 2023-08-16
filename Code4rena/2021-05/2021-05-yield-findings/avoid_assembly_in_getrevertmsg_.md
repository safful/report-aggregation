## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Avoid assembly in getRevertMsg ](https://github.com/code-423n4/2021-05-yield-findings/issues/15) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function getRevertMsg of RevertMsgExtractor.sol uses assembly to retrieve revert information.
The latest solidity version have new functions that allows you to retrieve information without assembly.

## Proof of Concept
// https://github.com/code-423n4/2021-05-yield/blob/main/contracts/utils/RevertMsgExtractor.sol
function getRevertMsg(bytes memory returnData) internal pure returns (string memory) {
..        assembly {
            // Slice the sighash.
            returnData := add(returnData, 0x04)
        }
 
## Tools Used

## Recommended Mitigation Steps
Below is a piece of code showing the new functionality:

pragma solidity ^0.8.1;

contract ContractError {
    function Underflow() public pure returns (uint) {
         uint x = 0;
         x--; // this will generate an underflow
         return x;
    }
    function UncheckedUnderflow() public pure returns (uint) {
         uint x = 0;
         unchecked { x--; } // this will generate an underflow
         return x;
    } 
}

contract C {
    ContractError e = new ContractError();
    
    function TestUnderflow() public view returns (string memory) {
         try e.Underflow() returns (uint) {
            return "Ok";
        } catch Error(string memory reason) {
            return reason;
        } catch Panic(uint _code) {
            if (_code == 0x01) { return "Assertion failed"; }
            else if (_code == 0x11) { return "Underflow/overflow"; }
            // We ignore the other errors.
            return "Other Panic";
        } catch (bytes memory reason) { 
            uint x=0;
            for (uint i=0;i<4;i++) //get first 4 bytes
                x = (x<<8) + uint(uint8(reason[i]));
        
            if (x == 0x08c379a0) // abi.encodeWithSignature("Error(string)")
                return "Error";
            return "Unknown";
        }
    }
}


