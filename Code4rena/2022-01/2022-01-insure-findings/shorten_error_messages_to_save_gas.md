## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Shorten Error Messages to Save Gas](https://github.com/code-423n4/2022-01-insure-findings/issues/87) 

# Handle

0xngndev


# Vulnerability details

## Impact

Error Messages that have a length of 32 or more one require one additional slot to be stored, causing extra gas costs when deploying the contract and when the function is executed and it reverts.

## Proof of Concept

I put together a quick proof to show the different impact of the errors we can have in Solidity:

- Long require errors => more than 32 bytes
- Short require errors => less than 32 bytes
- Custom errors

Here are the contract size findings:

```rust
//SPDX-License-Identifier: unlicensed
pragma solidity 0.8.10;

contract Errors {
  bool public thisIsFalse;
  error WithdrawalExceeded();

  /*
    Contract Size with just this function: 333 bytes;
  */
  function moreThan32Bytes() public {
    require(thisIsFalse, "ERROR: WITHDRAWAL_EXCEEDED_REQUEST");
  }

  /*
    Contract Size with just this function: 295 bytes;
//   */
  function lessThan32Bytes() public {
    require(thisIsFalse, "WITHDRAWAL_EXCEEDED_REQUEST");
  }

  /*
    Contract Size with just this function: 242 bytes;
  */
  function customError() public {
    if (!thisIsFalse) revert WithdrawalExceeded();
  }
}
```

I then run tests to see the gas costs of having the functions revert, and although these are not very accurate due to the fact that it’s hard to isolate the gas costs of a reverting function due to the order of execution (I can’t have an event that logs the gas before the function revert and another one after because the one after the revert will never be reached), it still shows some differences.

```rust
//SPDX-License-Identifier: unlicensed
pragma solidity 0.8.10;

import "ds-test/test.sol";
import "../Errors.sol";

contract ErrorsTest is DSTest {
  Errors errors;

  function setUp() public {
    errors = new Errors();
  }

  function testFailLessThan32Bytes() public logs_gas {
    errors.lessThan32Bytes();
  }

  function testFailMoreThan32Bytes() public logs_gas {
    errors.moreThan32Bytes();
  }

  function testFailCustomError() public logs_gas {
    errors.customError();
  }
}
```

```rust
Running 3 tests for "ErrorsTest.json":ErrorsTest
[PASS] testFailCustomError() (gas: 3161)
[PASS] testFailLessThan32Bytes() (gas: 3314)
[PASS] testFailMoreThan32Bytes() (gas: 3401)
```

## Tools Used

DappTools/Foundry

## Recommended Mitigation Steps

Personally, I would switch to custom errors and reverts to maximize the savings, but if you dislike revert syntax, then I would suggest to check which of your require errors have a length longer than 32, and shorten them so that their length is less than 32.

Here are some examples of the errors you could shorten in your `CDSTemplate.sol` contract:

- `ERROR: INITIALIZATION_BAD_CONDITIONS`
- `ERROR: WITHDRAWAL_NO_ACTIVE_REQUEST`
- `ERROR: WITHDRAWAL_EXCEEDED_REQUEST`

Removing the “ERROR” keyword should be enough for most of these. Bear in mind you can always have concise error messages and a section in your documentation that explains them further or have your natspec expand on them if you find them too cryptic. 

An example of how to apply a custom error in the first error would be to just have the error `say BadConditions()`. The user knows it’s an error because the function call failed, and the user knows it has happened in the initialize function because he called it, so `BadConditions()` should be a clear message despite being concise

