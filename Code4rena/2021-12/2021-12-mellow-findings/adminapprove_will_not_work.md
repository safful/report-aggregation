## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [adminApprove will not work](https://github.com/code-423n4/2021-12-mellow-findings/issues/117) 

# Handle

pauliax


# Vulnerability details

## Impact
function adminApprove intends to allow an admin to approve NFTs on behalf of users:
```solidity
  function adminApprove(address newAddress, uint256 nft) external {
    require(_isProtocolAdmin(_msgSender()), ExceptionsLibrary.ADMIN);
    IERC721(address(this)).approve(newAddress, nft);
  }
```

However, when it calls .approve, it will check the ownership again, so only the calls from admin and owner/approved will pass: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol#L116-L119

This makes this function ineffective.

## Recommended Mitigation Steps
Based on my understanding, it should call ._approve(...).

