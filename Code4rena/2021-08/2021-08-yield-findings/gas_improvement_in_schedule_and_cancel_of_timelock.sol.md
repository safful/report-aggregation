## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- Timelock

# [gas improvement in schedule and cancel of TimeLock.sol](https://github.com/code-423n4/2021-08-yield-findings/issues/3) 

# Handle

gpersoon


# Vulnerability details

## Impact
The functions schedule and cancel of TimeLock.sol receive the parameters targets and data.
This is not absolutely necessary. Receiving txHash would be enough, as txHash is verified in the function execute.
This is like a commit and reveal scheme.
It would save some gas and contract complexity.

This assumes that the parameters  targets and data are accessible offchain for verification, which I would think would be true anyway.

## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.

## Tools Used

## Recommended Mitigation Steps
You can use the following approach if you want to save some gas.
Note: the parameters targets and data won't be available onchain in events (at least until execute has been performed)

 function schedule(bytes32 txHash, uint256 eta) external override auth {
        require(eta >= block.timestamp + delay, "Must satisfy delay."); // This also prevents setting eta = 0 and messing up the state        
        require(transactions[txHash] == 0, "Transaction not unknown.");
        transactions[txHash] = eta;        
       ....
    }

    function cancel(bytes32 txHash) external override auth {
        require(transactions[txHash] != 0, "Transaction hasn't been scheduled.");
        delete transactions[txHash];
       ....
    }

