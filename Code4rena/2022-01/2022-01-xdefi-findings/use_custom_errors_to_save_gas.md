## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Use Custom Errors to save Gas](https://github.com/code-423n4/2022-01-xdefi-findings/issues/22) 

# Handle

Dravee


# Vulnerability details

## Impact  
Custom errors from Solidity 0.8.4 are cheaper than revert strings.
  
## Proof of Concept  
Source: https://blog.soliditylang.org/2021/04/21/custom-errors/:

Starting from [Solidity v0.8.4](https://github.com/ethereum/solidity/releases/tag/v0.8.4), there is a convenient and gas-efficient way to explain to users why an operation failed through the use of custom errors. Until now, you could already use strings to give more information about failures (e.g., `revert("Insufficient funds.");`), but they are rather expensive, especially when it comes to deploy cost, and it is difficult to use dynamic information in them.

Custom errors are defined using the `error` statement, which can be used inside and outside of contracts (including interfaces and libraries).

Instances include:  
```  
XDEFIDistribution.sol:40:        require((XDEFI = XDEFI_) != address(0), "INVALID_TOKEN");
XDEFIDistribution.sol:47:        require(owner == msg.sender, "NOT_OWNER");
XDEFIDistribution.sol:52:        require(_locked == 0, "LOCKED");
XDEFIDistribution.sol:63:        require(pendingOwner == msg.sender, "NOT_PENDING_OWNER");
XDEFIDistribution.sol:82:            require(duration <= uint256(18250 days), "INVALID_DURATION");
XDEFIDistribution.sol:115:        require(lockAmount_ <= amountUnlocked_, "INSUFFICIENT_AMOUNT_UNLOCKED");
XDEFIDistribution.sol:145:        require(totalUnitsCached > uint256(0), "NO_UNIT_SUPPLY");
XDEFIDistribution.sol:170:        require(lockAmount_ <= amountUnlocked_, "INSUFFICIENT_AMOUNT_UNLOCKED");
XDEFIDistribution.sol:207:        require(count > uint256(1), "MIN_2_TO_MERGE");
XDEFIDistribution.sol:214:            require(ownerOf(tokenId) == msg.sender, "NOT_OWNER");
XDEFIDistribution.sol:215:            require(positionOf[tokenId].expiry == uint32(0), "POSITION_NOT_UNLOCKED"); 
XDEFIDistribution.sol:227:        require(_exists(tokenId_), "NO_TOKEN");
XDEFIDistribution.sol:232:        require(_exists(tokenId_), "NO_TOKEN");
XDEFIDistribution.sol:255:        require(amount_ != uint256(0) && amount_ <= MAX_TOTAL_XDEFI_SUPPLY, "INVALID_AMOUNT");
XDEFIDistribution.sol:259:        require(bonusMultiplier != uint8(0), "INVALID_DURATION");
XDEFIDistribution.sol:295:        require(ownerOf(tokenId_) == account_, "NOT_OWNER");
XDEFIDistribution.sol:304:        require(expiry != uint32(0), "NO_LOCKED_POSITION");
XDEFIDistribution.sol:305:        require(block.timestamp >= uint256(expiry), "CANNOT_UNLOCK");
XDEFIDistribution.sol:322:        require(count > uint256(1), "USE_UNLOCK");
```  
  
## Tools Used  
VS Code
  
## Recommended Mitigation Steps  
Replace revert strings with custom errors.


