## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Double update on storage pointer can lead to massive gas consumption](https://github.com/code-423n4/2021-11-fairside-findings/issues/34) 

# Handle

rfa


# Vulnerability details

## Impact
When you are reading a value from a storage and using a storage pointer instead of memory, you write directly to the storage instead of the memory. 
In the https://github.com/code-423n4/2021-11-fairside/blob/main/contracts/network/FSDNetwork.sol#L234 this line is reading membership[msg.sender] with a storage pointer, 
this means any changes to the user variable, is updating directly to the membership[msg.sender], 
therefore https://github.com/code-423n4/2021-11-fairside/blob/main/contracts/network/FSDNetwork.sol#L294 this line update, makes it useless since the data already written to the membership[msg.sender]

## Proof of Concept
 https://github.com/code-423n4/2021-11-fairside/blob/main/contracts/network/FSDNetwork.sol#L234-L294


## Tools Used

## Recommended Mitigation Steps
 https://github.com/code-423n4/2021-11-fairside/blob/main/contracts/network/FSDNetwork.sol#L294 membership[msg.sender] = user;

