## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`lending-market/Note.sol` Wrong implementation of access control](https://github.com/code-423n4/2022-06-canto-findings/issues/173) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/b93e2867a64b420ce6ce317f01c7834a7b6b17ca/contracts/Note.sol#L13-L31


# Vulnerability details

```solidity
function _mint_to_Accountant(address accountantDelegator) external {
    if (accountant == address(0)) {
        _setAccountantAddress(msg.sender);
    }
    require(msg.sender == accountant, "Note::_mint_to_Accountant: ");
    _mint(msg.sender, type(uint).max);
}

function RetAccountant() public view returns(address) {
    return accountant;
}

function _setAccountantAddress(address accountant_) internal {
    if(accountant != address(0)) {
        require(msg.sender == admin, "Note::_setAccountantAddress: Only admin may call this function");
    }
    accountant = accountant_;
    admin = accountant;
}
```

`_mint_to_Accountant()` calls `_setAccountantAddress()` when `accountant == address(0)`, which will always be the case when `_mint_to_Accountant()` is called for the first time.

And `_setAccountantAddress()` only checks if `msg.sender == admin` when `accountant != address(0)` which will always be `false`, therefore the access control is not working.

L17 will then check if `msg.sender == accountant`, now it will always be the case, because at L29, `accountant` was set to `msg.sender`.

