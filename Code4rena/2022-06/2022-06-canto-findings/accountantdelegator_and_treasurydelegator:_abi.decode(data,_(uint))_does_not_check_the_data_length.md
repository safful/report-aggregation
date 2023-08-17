## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [AccountantDelegator and TreasuryDelegator: abi.decode(data, (uint)) does not check the data length](https://github.com/code-423n4/2022-06-canto-findings/issues/84) 

# Lines of code

https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Treasury/TreasuryDelegator.sol#L44-L56


# Vulnerability details

## Impact
In AccountantDelegator and TreasuryDelegator contracts, when using abi.decode(data, (uint)) to convert data to uint type, the length of data is not checked, when the returned data is of bytes type, the abi.decode will return 0x20.
## Proof of Concept
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Treasury/TreasuryDelegator.sol#L44-L56
https://github.com/Plex-Engineer/lending-market/blob/755424c1f9ab3f9f0408443e6606f94e4f08a990/contracts/Accountant/AccountantDelegator.sol#L54-L74
This contract can test that when the function returns bytes data, abi.encode will decode the return value as 0x20.
```
pragma solidity 0.8.10;
contract A{
    uint public destination;
    uint256 public number;
    function convertA() external{
        (bool su,bytes memory ret )= address(this).call(abi.encodeWithSelector(this.ret.selector));
        number = ret.length;
        destination = abi.decode(ret, (uint));
    }
    function ret() public returns(bytes memory){
        return "1234";
    }
}
```
## Tools Used
None
## Recommended Mitigation Steps
Requires data.length == 32

