## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas: Inlining logic that's used only once in the contract](https://github.com/code-423n4/2021-12-sublime-findings/issues/40) 

# Handle

0xngndev


# Vulnerability details

### Impact

Reduce code size and gas expenditure by inlining internal functions that are only used once throughout your contract.

In some of your contracts, like `AaveYield.sol`, you have an `initialize` function that calls the internal functions `updateSavingsAccount` and `updateAaveAddresses`. These two functions are only called in the `initialize` function, so you can save some gas and code size by simply inlining their logic inside `initialize`. **The tradeoff, of course, is that you lose readability, and because `initialize` will only be called once, perhaps it's not worth the tradeoff.**

To test the difference in code size and gas expenditure I wrote a example contract replicating the behaviour to see how much cheaper inlining the logic was.

**The results:**

Inlining the logic:

- Gas spent: 45155
- Code size: 453 bytes

Separating into internal functions:

- Gas spent: 45189
- Code size: 467 bytes

### Proof of Concept

Contracts I used to test the gas expenditure and code size differences:

- InliningLogic

```bash
//SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.10;

contract InliningLogic {
  address public someRandomAddress;
  address public anotherRandomAddress;

  function assignAddresses(
    address _someRandomAddress,
    address _anotherRandomAddress
  ) public {
    someRandomAddress = _someRandomAddress;
    anotherRandomAddress = _anotherRandomAddress;
  }
}
```

- InliningLogic.t —> DappTools test file

```bash
//SPDX-License-Identifier: unlicensed
pragma solidity 0.8.10;

import "ds-test/test.sol";
import "../InliningLogic.sol";

contract InliningLogicTest is DSTest {
  InliningLogic inliningLogicContract;

  function setUp() public {
    inliningLogicContract = new InliningLogic();
  }

  function testInliningLogic() public logs_gas {
    inliningLogicContract.assignAddresses(address(0x1), address(0x1));
  }
}
```

- NotInliningLogic

```bash
//SPDX-License-Identifier: unlicensed
pragma solidity 0.8.10;

contract NotInliningLogic {
  address public someRandomAddress;
  address public anotherRandomAddress;

  function assignAddresses(
    address _someRandomAddress,
    address _anotherRandomAddress
  ) public {
    _assignAddresses(_someRandomAddress, _anotherRandomAddress);
  }

  function _assignAddresses(
    address _someRandomAddress,
    address _anotherRandomAddress
  ) internal {
    someRandomAddress = _someRandomAddress;
    anotherRandomAddress = _anotherRandomAddress;
  }
}
```

- NotInliningLogic.t —> test file

```bash
//SPDX-License-Identifier: unlicensed
pragma solidity 0.8.10;

import "ds-test/test.sol";
import "../NotInliningLogic.sol";

contract NotInliningLogicTest is DSTest {
  NotInliningLogic notInliningLogicContract;

  function setUp() public {
    notInliningLogicContract = new NotInliningLogic();
  }

  function testNotInliningLogic() public logs_gas {
    notInliningLogicContract.assignAddresses(address(0x1), address(0x1));
  }
}
```

### Tools

DappTools

