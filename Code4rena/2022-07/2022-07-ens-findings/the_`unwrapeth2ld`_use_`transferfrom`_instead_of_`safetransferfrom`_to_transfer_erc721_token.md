## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- sponsor disputed

# [The `unwrapETH2LD` use `transferFrom` instead of `safeTransferFrom` to transfer ERC721 token](https://github.com/code-423n4/2022-07-ens-findings/issues/157) 

# Lines of code

https://github.com/code-423n4/2022-07-ens/blob/ff6e59b9415d0ead7daf31c2ed06e86d9061ae22/contracts/wrapper/NameWrapper.sol#L327-L346


# Vulnerability details

### Impact

The `unwrapETH2LD` use `transferFrom` to transfer ERC721 token, the `newRegistrant` could be an unprepared contract

### Proof of Concept

Should a ERC-721 compatible token be transferred to an unprepared contract, it would end up being locked up there. Moreover, if a contract explicitly wanted to reject ERC-721 safeTransfers.
Plus take a look to [the OZ safeTransfer comments](https://docs.openzeppelin.com/contracts/4.x/api/token/erc721#IERC721-transferFrom-address-address-uint256-);  
`Usage of this method is discouraged, use safeTransferFrom whenever possible.`

### Tools Used

Manual Review

### Recommended Mitigation Steps

```diff
    function unwrapETH2LD(
        bytes32 labelhash,
        address newRegistrant,
        address newController
    ) public override onlyTokenOwner(_makeNode(ETH_NODE, labelhash)) {
        _unwrap(_makeNode(ETH_NODE, labelhash), newController);
-       registrar.transferFrom(
+       registrar.safeTransferFrom(
            address(this),
            newRegistrant,
            uint256(labelhash)
        );
    }
```

