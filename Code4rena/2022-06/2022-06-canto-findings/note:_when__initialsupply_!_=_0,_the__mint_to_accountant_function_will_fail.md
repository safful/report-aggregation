## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Note: When _initialSupply ! = 0, the _mint_to_Accountant function will fail](https://github.com/code-423n4/2022-06-canto-findings/issues/125) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/ab31a612be354e252d72faead63d86b844172761/contracts/Note.sol#L13-L19


# Vulnerability details

## Impact
In Note contract, if _initialSupply ! = 0, _totalSupply will overflow when the _mint_to_Accountant function executes _mint(msg.sender, type(uint).max)
```
    constructor(string memory name_, string memory symbol_, uint256 totalSupply_) public {
        _name = name_;
        _symbol = symbol_;
	    _initialSupply = totalSupply_;
	    _totalSupply = totalSupply_;
    }
...
    function _mint(address account, uint256 amount) internal   {
        require(account != address(0), "ERC20: mint to the zero address");

        _beforeTokenTransfer(address(0), account, amount);

        _totalSupply += amount;
        _balances[account] += amount;
        emit Transfer(address(0), account, amount);

        _afterTokenTransfer(address(0), account, amount);
    }
```
## Proof of Concept
https://github.com/Plex-Engineer/lending-market/blob/ab31a612be354e252d72faead63d86b844172761/contracts/Note.sol#L13-L19
https://github.com/Plex-Engineer/lending-market/blob/ab31a612be354e252d72faead63d86b844172761/contracts/ERC20.sol#L29-L34
https://github.com/Plex-Engineer/lending-market/blob/ab31a612be354e252d72faead63d86b844172761/contracts/ERC20.sol#L237-L247
## Tools Used
None
## Recommended Mitigation Steps
ERC20.sol
```
    constructor(string memory name_, string memory symbol_) public {
        _name = name_;
        _symbol = symbol_;
    }
```
note.sol
```
    constructor() ERC20("Note", "NOTE") {
        admin = msg.sender;
    }
```

