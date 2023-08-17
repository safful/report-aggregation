## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Try-catch block at `Auction._createAuction()` will only catch string errors](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/240) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/auction/Auction.sol#L234


# Vulnerability details


The `_createAuction` function wraps the `token.mint()` call in a try-catch block, however this will only catch reverts that comes from the require keyword and not the reverts with custom errors or other kinds of errors (arithmetic over/underflow etc.)

## Impact
In case of an error at the `mint()` function the auction won't be settled till the owner intervenes and pauses the contract.

## Proof of Concept
Here's a test that proves that `catch Error()` doesn't catch custom errors (the test will fail):

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "forge-std/Test.sol";

contract ContractTest is Test {
    function testErr() public{
        Reverter r = new Reverter();
        try r.throwCustomError(){

        }catch Error(string memory) {

        }
    }
}

contract Reverter{
    error  MyErr();

    function throwCustomError() public{
        revert MyErr();
    }
}


```

## Recommended Mitigation Steps

Remove the `Error` so that it'll catch any kind of revert:

```diff
             // Pause the contract if token minting failed
-        } catch Error(string memory) {
+        } catch  {
             _pause();
         }
     }
```