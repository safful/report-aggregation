## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Not handling return value of transferFrom command can create inconsistency](https://github.com/code-423n4/2022-03-joyn-findings/issues/81) 

# Lines of code

https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/core-contracts/contracts/CoreCollection.sol#L175-L176
https://github.com/code-423n4/2022-03-joyn/blob/c9297ccd925ebb2c44dbc6eaa3effd8db5d2368a/core-contracts/contracts/ERC721Payable.sol#L54-L55


# Vulnerability details

The below transferFrom command is called at two places in the core contracts, followed by an emit event
```
payableToken.transferFrom(msg.sender,recipient,_amount)
emit ...(...);
```
The return value is not checked during the payableToken.transferFrom

## Impact
In the event of failure of payableToken.transferFrom(...), the emit event is still generated causing the downstream applications to capture
wrong transaction / state of the protocol.

## Proof of Concept
1. Contract CoreCollection.sol  
   function withdraw()

2. Contract ERC721Payable.sol
   function _handlePayment


## Recommended Mitigation Steps
Add a require statement as being used in the RoyaltyVault.sol
```
require( payableToken.transferFrom(msg.sender,recipient,_amount) == true,
            "Failed to transfer amount to recipient" );
```

