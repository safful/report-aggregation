## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Truncation in casting can lead to a founder receiving all the base tokens](https://github.com/code-423n4/2022-09-nouns-builder-findings/issues/303) 

# Lines of code

https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/token/Token.sol#L71-L126
https://github.com/code-423n4/2022-09-nouns-builder/blob/7e9fddbbacdd7d7812e912a369cfd862ee67dc03/src/token/Token.sol#L88


# Vulnerability details

## Impact
The initialize function of the `Token` contract receives an array of `FounderParams`, which contains the ownership percent of each founder as a `uint256`. The initialize function checks that the sum of the percents is not more than 100, but the value that is added to the sum of the percent is truncated to fit in `uint8`. This leads to an error because the value that is used for assigning the base tokens is the original, not truncated, `uint256` value.

This can lead to wrong assignment of the base tokens, and can also lead to a situation where not all the users will get the correct share of base tokens (if any).

## Proof of Concept
To verify this bug I created a foundry test. You can add it to the test folder and run it with `forge test --match-test testFounderGettingAllBaseTokensBug`.

This test deploys a token implementation and an `ERC1967` proxy that points to it, and initializes the proxy using an array of 2 founders, each having 256 ownership percent. The value which is added to the `totalOwnership` variable is a `uint8`, and when truncating 256 to fit in a `uint8` it will turn to 0, so this check will pass.

After the call to initialize, the test asserts that all the base token ids belongs to the first founder, which means the second founder didn't get any base tokens at all.

What actually happens here is that the first founder gets the first 256 token ids, and the second founder gets the next 256 token ids, but because the base token is calculated % 100, only the first 100 matters and they will be owned by the first owner.

This happens because `schedule`, which is equal to `100 / founderPct`, will be zero (`100 / 256 == 0` due to uint div operation), and the base token id won't be updated in `(baseTokenId += schedule) % 100` (this line contains another mistake, which will be reported in another finding). The place where it will be updated is in the `_getNextTokenId`, where it will be incremented by 1.

This exploit can work as long as the sum of the percents modulo 256 (truncation to `uint8`) is not more than 100.

```sol
// The relative path of this file is "test/FounderGettingAllBaseTokensBug.t.sol"

// SPDX-License-Identifier: MIT
pragma solidity 0.8.15;

import { Test } from "forge-std/Test.sol";

import { IManager } from "../src/manager/Manager.sol";
import { IToken, Token } from "../src/token/Token.sol";

import { TokenTypesV1 } from "../src/token/types/TokenTypesV1.sol";

import { ERC1967Proxy } from "../src/lib/proxy/ERC1967Proxy.sol";

contract FounderGettingAllBaseTokensBug is Test, TokenTypesV1 {

    Token imp;
    address proxy;
    
    function setUp() public virtual {
        // Deploying the implementation and the proxy
        imp = new Token(address(this));
        proxy = address(new ERC1967Proxy(address(imp), ""));
    }

    function testFounderGettingAllBaseTokensBug() public {

        IToken token = IToken(proxy);

        address chadFounder = address(0xdeadbeef);
        address betaFounder = address(0xBBBBBBBB); // beta

        // Creating 2 founders with `ownershipPct = 256`
        IManager.FounderParams[] memory founders = new IManager.FounderParams[](2);
        founders[0] = IManager.FounderParams({
            wallet: chadFounder,
            ownershipPct: 256,
            vestExpiry: 1 weeks
        });
        founders[1] = IManager.FounderParams({
            wallet: betaFounder,
            ownershipPct: 256,
            vestExpiry: 1 weeks
        });

        // Initializing the proxy with the founders data
        token.initialize(   
            founders, 
            // we don't care about these
            abi.encode("", "", "", "", ""),
            address(0),
            address(0)
        );

        // Asserting that the chad founder got all the base token ids
        // (`tokenId % 100` is calculated to get the base token, so it is enough to check only the first 100 token ids)
        for (uint i; i < 100; ++i) {
            assertEq(token.getScheduledRecipient(i).wallet == chadFounder, true);
        }

        // Run with `forge test --match-test testFounderGettingAllBaseTokensBug`
        // Results:
        //      [PASS] testFounderGettingAllBaseTokensBug() (gas: 13537465)
        // Great success
    }
```

## Tools Used
Manual audit & foundry for the PoC

## Recommended Mitigation Steps
Don't truncate the `founderPct` variable to a uint8 when adding it to the totalOwnership variable, or alternatively check that it is less than `type(uint8).max` (or less or equal to 100).
After applying this fix and running the test again, the result is:
```
[FAIL. Reason: INVALID_FOUNDER_OWNERSHIP()] testFounderGettingAllBaseTokensBug() (gas: 58674)
```