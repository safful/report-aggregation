## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-02

# [Multiple accounts can have the same identity](https://github.com/code-423n4/2023-01-canto-identity-findings/issues/177) 

# Lines of code

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/AddressRegistry.sol#L47


# Vulnerability details

Users can register their on-chain identity (ie their CID NFT) by calling `AddressRegistry.register()`

```solidity
File: src/AddressRegistry.sol
42:     function register(uint256 _cidNFTID) external {
43:         if (ERC721(cidNFT).ownerOf(_cidNFTID) != msg.sender)
44:             // We only guarantee that a CID NFT is owned by the user at the time of registration
45:             // ownerOf reverts if non-existing ID is provided
46:             revert NFTNotOwnedByUser(_cidNFTID, msg.sender);
47:         cidNFTs[msg.sender] = _cidNFTID;
48:         emit CIDNFTAdded(msg.sender, _cidNFTID);
49:     }
```

This overwrites `cidNFTs[msg.sender]` with the `cidNFTID` provided by the caller.

The issue is that there is nothing preventing several (2 or more) accounts to point to the same `cidNFTID`, ie have `cidNFTs[userA] == cidNFTs[userB]`

Note: the README mentioned that 
```
Transferring CID NFTs that are still referenced in the address registry: CID NFTs are transferrable on purpose and a user can transfer his CID NFT while it is still registered to his address if he wants to do so.
```

The issue described in this report is not that the CID NFT is transferrable, but that several accounts can point to the same CIDNFT id, which lead to several problems outlined below.


## Impact


Quoting the README:
```
Canto Identity NFTs (CID NFTs) represent individual on-chain identities
```


Here, several accounts can point to the same on-chain identity, breaking the requirement that the said identity should be **individual**

To illustrate the consequences of this, let us look at `CidNFT.add()`, which adds a new entry for the given subprotocol to the provided CID NFT:
- data is added by transferring a subprotocol NFT to the contract, which will write the NFT id in `cidData[_cidNFTID][_subprotocolName]`
- This NFT id represents traits that will be associated with the identity.

Because of the issue outlined above, the identity system can be abused: 

- Alice registers her CIDNft by calling `addressRegistry.register(N)`
- she transfers it to Bob, who then proceeds to call `addressRegistry.register(N)` to register it.
- at this point, `cidNFT` of id `N` points to both Alice and Bob: `addressRegistry.getCID(Alice) == addressRegistry.getCID(Bob)`
- Bob calls `CidNFT.add()` to add a subProtocol NFT X to his identity `N` . Because Alice is also associated to the `CIDNFT` `N`, she essentially added this trait for free (assuming subprotocols will monetize their tokens, Bob had to pay the cost of the subProtocol NFT X, but Alice did not).
- This can also have further consequences depending on what can be done with these traits (e.g: a protocol giving rewards for users with a trait of the subProtocol NFT X, Bob could be front run by Alice and not receive a reward he was entitled to)

Overall, because this issue impacts a key aspect of the protocol (identities are not individual) and can lead to a form of `theft` in certain conditions (in the scenario above, Alice got a trait added to her identity for "free"), the Medium severity seems appropriate.


## Proof Of Concept
This test shows how two users can point to the same `CID`.
Add it to `AddressRegistry.t.sol`

```solidity
function testTwoUsersSameCID() public {
    uint256 nftIdOne = 1;
    address Alice = users[0];
    address Bob = users[1];

    // 1 - Alice mints NFT
    vm.startPrank(Alice);
    bytes[] memory addList;
    cidNFT.mint(addList);
    assertEq(cidNFT.ownerOf(nftIdOne), Alice);

    // 2 - Alice registers the NFT
    addressRegistry.register(nftIdOne);

    // 3 - Alice transfers the CID NFT to Bob
    cidNFT.transferFrom(Alice, Bob, nftIdOne);
    vm.stopPrank();

    // 4 - Bob registers the nft
    vm.startPrank(Bob);
    addressRegistry.register(nftIdOne);

    // 5 - Alice and Bob have the same identity
    uint256 cidAlice = addressRegistry.getCID(Alice);
    uint256 cidBob = addressRegistry.getCID(Bob);
    assertEq(cidAlice, cidBob);
}
```


## Tools Used

Manual Analysis, Foundry

## Mitigation

`AddressRegistry` should have an additional mapping to track the account associated with a given `cifNTFID`.

```diff
File: src/AddressRegistry.sol
20:     /// @notice Stores the mappings of users to their CID NFT
21:     mapping(address => uint256) private cidNFTs;
+       mapping(uint256 => address) private accounts;
```

When registering, the code would check if the `cidNFTID` has an account associated with it.
If that is the case, `cidNFTs` for this user would be set to 0, preventing several users from having the same identity.

```diff
File: src/AddressRegistry.sol
42: function register(uint256 _cidNFTID) external {
43:         if (ERC721(cidNFT).ownerOf(_cidNFTID) != msg.sender)
44:             // We only guarantee that a CID NFT is owned by the user at the time of registration
45:             // ownerOf reverts if non-existing ID is provided
46:             revert NFTNotOwnedByUser(_cidNFTID, msg.sender);
+           if (accounts[_cidNFTID] != address(0)) {
+                 delete cidNFTs[accounts[_cidNFTID]];
+                 emit CIDNFTRemoved(accounts[_cidNFTID], _cidNFTID);
+}
47:         cidNFTs[msg.sender] = _cidNFTID;
+           accounts[_cidNFTID] = msg.sender;
48:         emit CIDNFTAdded(msg.sender, _cidNFTID);
49:     }
```