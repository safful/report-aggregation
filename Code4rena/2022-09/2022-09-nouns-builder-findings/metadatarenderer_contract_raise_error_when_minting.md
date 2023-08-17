## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [MetadataRenderer contract raise error when minting](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/69) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/token/metadata/MetadataRenderer.sol#L194


# Vulnerability details

## Impact
It is not possible to mint a ERC721 token if its properties has different length than it's items.

## Proof of Concept

I run the following test to reproduce the error:

        deployMock();

        vm.prank(address(governor));
        string[] memory _names = new string[](1);
        _names[0] = "propertyName"; //fill _names with some value
        MetadataRendererTypesV1.ItemParam[] memory _items; //define empty array
        
        MetadataRendererTypesV1.IPFSGroup memory _ipfsGroup; 
        _ipfsGroup.baseUri = "";
        _ipfsGroup.extension = "";
        MetadataRenderer(token.metadataRenderer()).addProperties(_names, _items, _ipfsGroup); //call add property with _items array empty.

        vm.stopPrank();
        vm.prank(address(auction));

        uint256 success = token.mint();//error happens inside here
        assert(success != 0);

        vm.stopPrank();
        

Log from Foundry console:

    ├─ [736] TOKEN::metadataRenderer() [staticcall]
    │   ├─ [353] Token::metadataRenderer() [delegatecall]
    │   │   └─ ← METADATA_RENDERER: [0x7076fd06ec2d09d4679d9c35a8db81ace7a07ee2]
    │   └─ ← METADATA_RENDERER: [0x7076fd06ec2d09d4679d9c35a8db81ace7a07ee2]
    ├─ [78618] METADATA_RENDERER::addProperties(["propertyName"], [], ("", ""))
    │   ├─ [78172] MetadataRenderer::addProperties(["propertyName"], [], ("", "")) [delegatecall]
    │   │   ├─ emit OwnerUpdated(prevOwner: FOUNDER: [0xd3562fd10840f6ba56112927f7996b7c16edfcc1], newOwner: TREASURY: [0xf8cf955543f1ce957b81c1786be64d5fc96ad7b5])      
    │   │   ├─ emit PropertyAdded(id: 0, name: "propertyName")
    │   │   └─ ← ()
    │   └─ ← ()
    ├─ [0] VM::stopPrank()
    │   └─ ← ()
    ├─ [0] VM::prank(AUCTION: [0x9a1450e42d752b8731bc88f20dbaa9154642f1e6])
    │   └─ ← ()
    ├─ [121037] TOKEN::mint()
    │   ├─ [120650] Token::mint() [delegatecall]
    │   │   ├─ emit Transfer(from: 0x0000000000000000000000000000000000000000, to: FOUNDER: [0xd3562fd10840f6ba56112927f7996b7c16edfcc1], tokenId: 0)
    │   │   ├─ emit DelegateVotesChanged(delegate: FOUNDER: [0xd3562fd10840f6ba56112927f7996b7c16edfcc1], prevTotalVotes: 0, newTotalVotes: 1)
    │   │   ├─ [25762] METADATA_RENDERER::onMinted(0)
    │   │   │   ├─ [25372] MetadataRenderer::onMinted(0) [delegatecall]
    │   │   │   │   └─ ← "Division or modulo by 0"
    │   │   │   └─ ← "Division or modulo by 0"
    │   │   └─ ← "Division or modulo by 0"
    │   └─ ← "Division or modulo by 0"
    └─ ← "Division or modulo by 0"



## Tools Used
Foundry
Manual

## Recommended Mitigation Steps
It could be mitigated checking length of both arrays in MetadataRenderer.addProperties() method.

It could be done after those lines:
https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/token/metadata/MetadataRenderer.sol#L111-L115

Also I recommend to move those declaration and new validation at the beginning to save gas.