## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Unlinked address can link immediately again](https://github.com/code-423n4/2021-12-sublime-findings/issues/54) 

# Handle

gpersoon


# Vulnerability details

## Impact
After a master calls unlinkAddress() to unlink an address, the address that has just been unlinked can directly link again without permission.
The address that is just unlinked can call linkAddress(masterAddress) which will execute because pendingLinkAddresses is still set.
Assuming the master has unlinked for a good reason it is unwanted to be able to be linked again without any permission from the master.

Note: a master can prevent this by calling cancelAddressLinkingRequest(), but this doesn't seem logical to do

## Proof of Concept
https://github.com/code-423n4/2021-12-sublime/blob/e688bd6cd3df7fefa3be092529b4e2d013219625/contracts/Verification/Verification.sol#L129-L154

```JS
    function unlinkAddress(address _linkedAddress) external {
        address _linkedTo = linkedAddresses[_linkedAddress].masterAddress;
        require(_linkedTo != address(0), 'V:UA-Address not linked');
        require(_linkedTo == msg.sender, 'V:UA-Not linked to sender');
        delete linkedAddresses[_linkedAddress]; 
       ...
}
    function linkAddress(address _masterAddress) external {
        require(linkedAddresses[msg.sender].masterAddress == address(0), 'V:LA-Address already linked');   // == true (after unlinkAddress)
        require(pendingLinkAddresses[msg.sender][_masterAddress], 'V:LA-No pending request');                 // == true (after unlinkAddress)
        _linkAddress(msg.sender, _masterAddress);                                                                                           // // pendingLinkAddresses not reset
    }

function cancelAddressLinkingRequest(address _linkedAddress) external {
        ... 
        delete pendingLinkAddresses[_linkedAddress][msg.sender]; // only location where pendingLinkAddresses is reset
```

## Tools Used

## Recommended Mitigation Steps
Add something like to following at the end of linkAddress:
```JS
 delete pendingLinkAddresses[msg.sender][_masterAddress]; 
```

