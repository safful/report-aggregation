## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-13

# [Ownership of EscherERC721.sol contracts can be changed, thus creator roles become useless ](https://github.com/code-423n4/2022-12-escher-findings/issues/521) 

# Lines of code

https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher.sol#L11
https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/65420cb9c943c32eb7e8c9da60183a413d90067a/contracts/access/AccessControlUpgradeable.sol#L150
https://github.com/code-423n4/2022-12-escher/blob/main/src/Escher721Factory.sol#L32


# Vulnerability details

## Impact
(
    creator = has a CREATOR_ROLE in Escher.sol
    non-creator = doesn't have a CREATOR_ROLE in Escher.sol
)

Currently creating an ERC721 edition via the ```Escher721Factory.sol``` contract requires a user to have the ``` CREATOR_ROLE ``` in the main ``` Escher.sol ``` contract.
This requirement would mean that only users with the aforementioned role can be admins of editions. This requirement can be bypassed by having a 'malicious' creator
create an edition for someone who doesn't have the  ``` CREATOR_ROLE ``` set by creating the edition and granting the ```DEFAULT_ADMIN_ROLE``` to the non-creator via AccessControl.sol's ```grantRole()``` function. This way the non-creator can revoke the original creator's roles in this edition and gain full ownership. Now this non-creator admin can create sales and operate as if he/she was a creator. 

This defeats the point of having a role for creators and makes this function of the protocol not as described == faulty. 

## Proof of Concept
A creator can benefit from his role by taking in payments for creating ERC721 editions for other people. This would make sense so that his risk can be covered. 

1. A creator gets onboarded to Escher. 
2. For some time he stays good but then people start offering payments for edition ownership
3. The more creators there are in Escher the less of a chance to get caught but then again the more inclusive Escher gets the more people will pay to get their own edition,
    which makes this pretty dangerous
3. This creator creates an edition with the payer's inputs and grants the payer the DEFAULT_ADMIN_ROLE
4. Payer revokes all of the creator's roles and becomes the new admin 

You can edit the Escher721.t.sol file to look like this and then run the test normally, everything should go through without errors: 
``` 
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

import "forge-std/Test.sol";
import {EscherTest} from "./utils/EscherTest.sol";

contract Escher721Test is EscherTest {
    function setUp() public override {
        super.setUp();
    }
    function test_grantRoles() public {
        address _this = address(this);
        // Malicious creator grants someone else the rights for this edition
        edition.grantRole(bytes32(0x0), address(9));
        vm.prank(address(9));
        // Now this user can grant/revoke roles
        edition.grantRole(edition.MINTER_ROLE(), address(9));
        assertEq(edition.hasRole(bytes32(0x0), address(9)), true);
        // clean out the partner
        edition.revokeRole(bytes32(0x0), _this);
        assertEq(edition.hasRole(bytes32(0x0), _this), false);
    }
}
```

This kind of attack/abuse is currently hard to track. There is no centralized database of created editions and their admins at the time of creations (i.e. a mapping). This makes it hard to track down malicious creators who create editions for other people. Looping through the emitted events and comparing current admins to the emitted admins is a hassle especially if this protocol gains a lot of traction in the future which I assume is the end goal here. 

## Tools Used
Manual review, visual studio code, forge

## Recommended Mitigation Steps

In ```EscherERC721.sol``` implementation contract, it is recommended to override the ```grantRole()``` function of ```AccessControlUpgradeable.sol``` with something like:
```
function grantRole(bytes32 role, address account) internal override {
    revert("Admins can't be chagned");
}
```

This will disable the granting of roles after initialization. The initialization function already has the required granting of roles done and they cannot be changed after this fix.

Overall it would be recommended to store the created editions in a mapping for example to prevent problems like these.
