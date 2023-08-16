## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Incorrect implementation of Lender can result in lost tokens](https://github.com/code-423n4/2022-03-maple-findings/issues/16) 

# Lines of code

https://github.com/maple-labs/loan/blob/main/contracts/MapleLoanInternals.sol#L332-L344


# Vulnerability details

## Impact
MapleLoanInternals._sendFee should check returnData.length == 32 before decoding, otherwise if it returns bytes data, the abi.decode will return 0x20, result in lost tokens.
## Proof of Concept
https://github.com/maple-labs/loan/blob/main/contracts/MapleLoanInternals.sol#L332-L344
This contract can test that when the function returns bytes data, abi.encode will decode the return value as 0x20.
```
pragma solidity 0.8.7;
contract A{
    address public destination;
    uint256 public number;
    function convertA() external{
        (bool su,bytes memory ret )= address(this).call(abi.encodeWithSelector(this.ret.selector));
        number = ret.length;
        destination = abi.decode(ret, (address));
    }
    function ret() public returns(bytes memory){
        return "0x74d754378a59Ab45d3E6CaC83f0b87E8E8719270";
    }
}
```
## Tools Used
None
## Recommended Mitigation Steps
```
    function _sendFee(address lookup_, bytes4 selector_, uint256 amount_) internal returns (bool success_) {
        if (amount_ == uint256(0)) return true;

        ( bool success , bytes memory data ) = lookup_.call(abi.encodeWithSelector(selector_));

+       if (!success || data.length != uint256(32)) return false;

        address destination = abi.decode(data, (address));

        if (destination == address(0)) return false;

        return ERC20Helper.transfer(_fundsAsset, destination, amount_);
    }
```


