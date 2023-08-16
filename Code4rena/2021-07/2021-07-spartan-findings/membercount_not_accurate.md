## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- disagree with severity

# [memberCount not accurate](https://github.com/code-423n4/2021-07-spartan-findings/issues/26) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function depositForMember of BondVault.sol adds user to the array arrayMembers.
However it does this for each asset that a user deposits. Suppose a user deposit multiple assets, than the user is added multiple times to the array arrayMembers.

This will mean the memberCount() doesn't show accurate results.
Also allMembers() will contain duplicate members

## Proof of Concept
// https://github.com/code-423n4/2021-07-spartan/blob/main/contracts/BondVault.sol#L60
function depositForMember(address asset, address member, uint LPS) external onlyDAO returns(bool){
        if(!mapBondAsset_memberDetails[asset].isMember[member]){
            mapBondAsset_memberDetails[asset].isMember[member] = true; // Register user as member (scope: user -> asset)
            arrayMembers.push(member); // Add user to member array (scope: vault)
            mapBondAsset_memberDetails[asset].members.push(member); // Add user to member array (scope: user -> asset)
        }
       ...

    // Get the total count of all existing & past BondVault members
    function memberCount() external view returns (uint256 count){
        return arrayMembers.length;
    }
    function allMembers() external view returns (address[] memory _allMembers){
        return arrayMembers;
    }

## Tools Used

## Recommended Mitigation Steps
Use a construction like this:
mapping(address => bool) isMember;
   if(!isMember[member]){
            isMember[member] = true;
            arrayMembers.push(member); 
   }
            

