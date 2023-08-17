## Tags

- bug
- duplicate
- 2 (Med Risk)
- sponsor confirmed
- selected for report
- responded

# [It is possible that operator loses sent ETH after calling `HolographOperator` contract's `executeJob` function](https://github.com/code-423n4/2022-10-holograph-findings/issues/418) 

# Lines of code

https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/HolographOperator.sol#L301-L439
https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/HolographOperator.sol#L445-L478


# Vulnerability details

## Impact
ETH can be sent when calling the `HolographOperator` contract's `executeJob` function, which can execute the following code.
```solidity
File: contracts\HolographOperator.sol
419:     try
420:       HolographOperatorInterface(address(this)).nonRevertingBridgeCall{value: msg.value}(
421:         msg.sender,
422:         bridgeInRequestPayload
423:       )
424:     {
425:       /// @dev do nothing
426:     } catch {
427:       _failedJobs[hash] = true;
428:       emit FailedOperatorJob(hash);
429:     }
```

Executing the `try ... {...} catch {...}` code mentioned above will execute `HolographOperatorInterface(address(this)).nonRevertingBridgeCall{value: msg.value}(...)`. Calling the `nonRevertingBridgeCall` function can possibly execute `revert(0, 0)` if the external call to the bridge contract is not successful. When this occurs, the code in the `catch` block of the `try ... {...} catch {...}` code mentioned above will run, which does not make calling the `executeJob` function revert. As a result, even though the job is not successfully executed, the sent ETH is locked in the `HolographOperator` contract since there is no other way to transfer such sent ETH out from this contract. In this situation, the operator that calls the `executeJob` function will lose the sent ETH.

https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/HolographOperator.sol#L301-L439
```solidity
  function executeJob(bytes calldata bridgeInRequestPayload) external payable {
    
    ...

    /**
     * @dev execute the job
     */
    try
      HolographOperatorInterface(address(this)).nonRevertingBridgeCall{value: msg.value}(
        msg.sender,
        bridgeInRequestPayload
      )
    {
      /// @dev do nothing
    } catch {
      _failedJobs[hash] = true;
      emit FailedOperatorJob(hash);
    }
    /**
     * @dev every executed job (even if failed) increments total message counter by one
     */
    ++_inboundMessageCounter;
    /**
     * @dev reward operator (with HLG) for executing the job
     * @dev this is out of scope and is purposefully omitted from code
     */
    ////  _bondedOperators[msg.sender] += reward;
  }
```

https://github.com/code-423n4/2022-10-holograph/blob/main/contracts/HolographOperator.sol#L445-L478
```solidity
  function nonRevertingBridgeCall(address msgSender, bytes calldata payload) external payable {
    require(msg.sender == address(this), "HOLOGRAPH: operator only call");
    assembly {
      /**
       * @dev remove gas price from end
       */
      calldatacopy(0, payload.offset, sub(payload.length, 0x20))
      /**
       * @dev hToken recipient is injected right before making the call
       */
      mstore(0x84, msgSender)
      /**
       * @dev make non-reverting call
       */
      let result := call(
        /// @dev gas limit is retrieved from last 32 bytes of payload in-memory value
        mload(sub(payload.length, 0x40)),
        /// @dev destination is bridge contract
        sload(_bridgeSlot),
        /// @dev any value is passed along
        callvalue(),
        /// @dev data is retrieved from 0 index memory position
        0,
        /// @dev everything except for last 32 bytes (gas limit) is sent
        sub(payload.length, 0x40),
        0,
        0
      )
      if eq(result, 0) {
        revert(0, 0)
      }
      return(0, 0)
    }
  }
```

## Proof of Concept
First, please add the following `OperatorAndBridgeMocks.sol` file in `src\mock\`.
```solidity
pragma solidity 0.8.13;

// OperatorMock contract simulates the logic flows used in HolographOperator contract's executeJob and nonRevertingBridgeCall functions
contract OperatorMock {
    bool public isJobExecuted = true;

    BridgeMock bridgeMock = new BridgeMock();

    // testExecuteJob function here simulates the logic flow used in HolographOperator.executeJob function
    function testExecuteJob() external payable {
        try IOperatorMock(address(this)).testBridgeCall{value: msg.value}() {
        } catch {
            isJobExecuted = false;
        }
    }
    
    // testBridgeCall function here simulates the logic flow used in HolographOperator.nonRevertingBridgeCall function
    function testBridgeCall() external payable {
        // as a simulation, the external call that sends ETH to bridgeMock contract will revert
        (bool success, ) = address(bridgeMock).call{value: msg.value}("");
        if (!success) {
            assembly {
                revert(0, 0)
            }
        }
        assembly {
            return(0, 0)
        }
    }
}

interface IOperatorMock {
    function testBridgeCall() external payable;
}

contract BridgeMock {
    receive() external payable {
        revert();
    }
}
```

Then, please add the following `POC.ts` file in `test\`.
```typescript
import { expect } from "chai";
import { ethers } from "hardhat";

describe('POC', () => {
    it("It is possible that operator loses sent ETH after calling HolographOperator contract's executeJob function", async () => {
        // deploy operatorMock contract that simulates
        //   the logic flows used in HolographOperator contract's executeJob and nonRevertingBridgeCall functions
        const OperatorMockFactory = await ethers.getContractFactory('OperatorMock');
        const operatorMock = await OperatorMockFactory.deploy();
        await operatorMock.deployed();

        await operatorMock.testExecuteJob({value: 500});

        // even though the job is not successfully executed, the sent ETH is locked in operatorMock contract
        const isJobExecuted = await operatorMock.isJobExecuted();
        expect(isJobExecuted).to.be.eq(false);
        expect(await ethers.provider.getBalance(operatorMock.address)).to.be.eq(500);
    });
});
```

Last, please run `npx hardhat test test/POC.ts --network hardhat`. The `It is possible that operator loses sent ETH after calling HolographOperator contract's executeJob function` test will pass to demonstrate the described scenario.

## Tools Used
VSCode

## Recommended Mitigation Steps
In the `catch` block of the `try ... {...} catch {...}` code mentioned above in the Impact section, the code can be updated to transfer the `msg.value` amount of ETH back to the operator, which is `msg.sender` for the `HolographOperator` contract's `executeJob` function, when this described situation occurs.