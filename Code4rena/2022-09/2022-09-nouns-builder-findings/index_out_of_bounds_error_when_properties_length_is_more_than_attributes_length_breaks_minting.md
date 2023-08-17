## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- edited-by-warden

# [Index out of bounds error when properties length is more than attributes length breaks minting](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/523) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/main/src/token/metadata/MetadataRenderer.sol#L188-L198


# Vulnerability details

## Description

When a token is minted, the ```MetadataRenderer.sol``` ```onMinted``` function is called which will set the particular token's attributes to a random item from one of the properties. A token has a maximum of 16 attributes, the first one being the total number of properties. The properties from which the token receives its attributes are supplied by the owner of the ```MetadataRenderer.sol``` contract by calling ```addProperties```. The issue is that the number of properties the owner can supply is not limited. If the number of properties is more than 15 then the ```onMinted``` function will revert due to the limit on the number of attributes a token may have.

## Impact

Since ```onMinted``` is always called when tokens are minted, the DAO will not be able to mint new tokens. There does not seem to be a way to remove properties so this would be unrecoverable.

## Proof of Concept

Test code added to ```Token.t.sol```:
```solidity
    function test_MetadataProperties() public {
        createUsers(2, 1 ether);

        address[] memory wallets = new address[](2);
        uint256[] memory percents = new uint256[](2);
        uint256[] memory vestExpirys = new uint256[](2);

        uint256 pct = 50;
        uint256 end = 4 weeks;

        unchecked {
            for (uint256 i; i < 2; ++i) {
                wallets[i] = otherUsers[i];
                percents[i] = pct;
                vestExpirys[i] = end;
            }
        }

        deployWithCustomFounders(wallets, percents, vestExpirys);

        // Check deployed correctly
        assertEq(token.totalFounders(), 2);
        assertEq(token.totalFounderOwnership(), 100);

        // Create 16 properties and items
        string[] memory names = new string[](16);
        MetadataRendererTypesV1.ItemParam[] memory items = new MetadataRendererTypesV1.ItemParam[](16);
        for (uint256 j; j < 16; j++) {
            names[j] = "aaa";                                                                               // Add random properties
            items[j].name = "aaa";                                                                          // Add random items
            items[j].propertyId = uint16(j);                                                                // Make sure all properties have items
            items[j].isNewProperty = true;
        }

        MetadataRendererTypesV1.IPFSGroup memory group = MetadataRendererTypesV1.IPFSGroup(
            "aaa",
            "aaa"
        );                                                                                                  // Add random IPFS group

        // Add 16 properties
        vm.prank(otherUsers[0]);
        metadataRenderer.addProperties(names, items, group);

        // Attempt to mint
        vm.prank(address(auction));
        vm.expectRevert(stdError.indexOOBError);
        token.mint();
    }
```

The test code above shows that the owner of ```MetadataRenderer.sol``` is able to add 16 properties with 1 items each. The ```auction``` contract is then unable to mint due to an "Index out of bounds" error.

Code from the ```onMinted``` function in ```MetadataRenderer.sol```:
```solidity
            // For each property:
            for (uint256 i = 0; i < numProperties; ++i) {
                // Get the number of items to choose from
                uint256 numItems = properties[i].items.length;

                // Use the token's seed to select an item
                tokenAttributes[i + 1] = uint16(seed % numItems);

                // Adjust the randomness
                seed >>= 16;
            }
```

The code above shows that when a token is minted and ```onMinted``` is called it will attempt to assign more than 16 attributes to the token which is not possible due to the ```tokenAttributes``` being limited to 16.

## Recommended Mitigation Steps

The maximum amount of properties an owner can add should be less than the maximum amount of attributes any token can have. Consider either limiting the ```properties``` variable in ```MetadataRenderer.sol``` to 15 or allow any number of attributes to be added to a token.