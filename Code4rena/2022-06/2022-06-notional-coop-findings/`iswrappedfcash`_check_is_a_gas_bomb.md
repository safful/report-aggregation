## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- Index

# [`IsWrappedFcash` check is a gas bomb](https://github.com/code-423n4/2022-06-notional-coop-findings/issues/188) 

# Lines of code

https://github.com/code-423n4/2022-06-notional-coop/blob/main/index-coop-notional-trade-module/contracts/protocol/modules/v1/NotionalTradeModule.sol#L639-L647


# Vulnerability details

## Impact
In the `_isWrappedFCash` check, the `notionalTradeModule` check whether the component is a wrappedCash with the following logic.

```soliditiy
        try IWrappedfCash(_fCashPosition).getDecodedID() returns(uint16 _currencyId, uint40 _maturity){
            try wrappedfCashFactory.computeAddress(_currencyId, _maturity) returns(address _computedAddress){
                return _fCashPosition == _computedAddress;
            } catch {
                return false;
            }
        } catch {
            return false;
        }
```

The above logic is dangerous when `_fCashPosition` do not revert on `getDecodedID` but instead give a wrong format of return value. The contract would try to decode the return value into `returns(uint16 _currencyId, uint40 _maturity)` and revert. The revert would consume what ever gas it's provided.

[CETH](https://etherscan.io/address/0x4Ddc2D193948926D02f9B1fE9e1daa0718270ED5) is an exmple.
There's a fallback function in `ceth`
```soliditiy
    function () external payable {
        requireNoError(mintInternal(msg.value), "mint failed");
    }
```
As a result, calling `getDecodedID` would not revert. Instead, calling `getDecodedID` of `CETH` would consume all remaining gas.
This creates so many issues. First, users would waste too much gas on a regular operation. Second, the transaction might fail if `ceth` is not the last position. Third, the wallet contract can not interact with set token with ceth as it consumes all gas.


## Proof of Concept

The following contract may fail to redeem setTokens as it consumes too much gas (with 20M gas limit).  

[Test.sol](https://gist.github.com/Jonah246/fad9e489fe84a6fb8b4894d7377fd8a2)

```soliditiy
    function test(uint256 _amount) external {
        cToken.approve(address(issueModule), uint256(-1));
        wfCash.approve(address(issueModule), uint256(-1));
        issueModule.issue(setToken, _amount, address(this));
        issueModule.redeem(setToken, _amount, address(this));
    }
```

Also, we can check how much gas it consumes with the following function.

```soliditiy
    function TestWrappedFCash(address _fCashPosition) public view returns(bool){
        if(!_fCashPosition.isContract()) {
            return false;
        }
        try IWrappedfCash(_fCashPosition).getDecodedID() returns(uint16 _currencyId, uint40 _maturity){
            try wrappedfCashFactory.computeAddress(_currencyId, _maturity) returns(address _computedAddress){
                return _fCashPosition == _computedAddress;
            } catch {
                return false;
            }
        } catch {
            return false;
        }
    }
```

Test this function with `cdai` and `ceth`, we can observe that there's huge difference of gas consumption here.
```
Gas used:            30376 of 130376
Gas used:            19479394 of 19788041
```
## Tools Used
Manual inspection. Hardhat

## Recommended Mitigation Steps

I recommend building a map in the notionalTradeModule and inserting the wrappeCash in the `mintFCashPosition` function.

```soliditiy
function addWrappedCash(uint16 _currencyId, uint40 _maturity) public {
    address computedAddress = wrappedfCashFactory.computeAddress(_currencyId, _maturity);
    wrappedFCash[computedAddress] = true;
}
```

Or we could replace the try-catch pattern with a low-level function call and check the return value's length before decoding it.

Something like this might be a fix.
```soliditiy
    (bool success, bytes memory returndata) = target.delegatecall(data);
    if (!success || returndata.length != DECODED_ID_RETURN_LENGTH) {
        return false;
    }
   // abi.decode ....
```

