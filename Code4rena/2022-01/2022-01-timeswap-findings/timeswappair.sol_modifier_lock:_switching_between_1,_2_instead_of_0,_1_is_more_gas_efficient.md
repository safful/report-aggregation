## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [TimeswapPair.sol modifier lock: Switching between 1, 2 instead of 0, 1 is more gas efficient](https://github.com/code-423n4/2022-01-timeswap-findings/issues/87) 

# Handle

bitbopper


# Vulnerability details

## Impact
`https://github.com/code-423n4/2022-01-timeswap/blob/5960e07d39f2b4a60cfabde1bd51f4b1e62e7e85/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L121:L126` could be more gas efficient

## Proof of Concept
### Version as in Repo

```
// SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity =0.8.4;

contract LockProofOld {
	uint256 locked = 0;

	modifier lock() {
	    require(locked == 0, 'E211');
	    locked = 1;
	    _;
	    locked = 0;
	}

	function test() public lock {
	}

}
```
### Proposed Version
```
// SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity =0.8.4;

contract LockProofNew {
	uint256 locked = 1;

	modifier lock() {
	    require(locked == 1, 'E211');
	    locked = 2;
	    _;
	    locked = 1;
	}

	function test() public lock {
	}

}
```

### Comparison
#### Test harness
```
// SPDX-License-Identifier: GPL-3.0-or-later
pragma solidity =0.8.4;

import "ds-test/test.sol";

import "./LockProofOld.sol";
import "./LockProofNew.sol";

contract LockProofTest is DSTest {
    LockProofOld lockproofold;
    LockProofNew lockproofnew;

    function setUp() public {
        lockproofold = new LockProofOld();
        lockproofnew = new LockProofNew();
    }

    function test_old() public {
		lockproofold.test();
    }

    function test_new() public {
		lockproofnew.test();
    }
}
```
#### Output
```
dapp test
Running 2 tests for src/LockProof.t.sol:LockProofTest
[PASS] test_old() (gas: 21042)
[PASS] test_new() (gas: 1136)
```


