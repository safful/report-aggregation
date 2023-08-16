## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Incorrect FeesSwept amount being emitted in sweepFees function](https://github.com/code-423n4/2021-10-tally-findings/issues/21) 

# Handle

harleythedog


# Vulnerability details

## Impact
In the sweepFees function, address(this).balance is being used as the "amount" in the SweepFees event immediately after a transfer. So, the amount in the event on line 258 will always be 0, but it should be what address(this).balance was before the transfer. This has implications on overall functionality, tools that are monitoring this event will receive incorrect information. A fix is to store the value before calling the transfer.

## Proof of Concept
referenced lines in sweepFees function: https://github.com/code-423n4/2021-10-tally/blob/main/contracts/swap/Swap.sol#:~:text=feeRecipient.transfer(address,this).balance%2C%20feeRecipient)%3B

A more correct implementation would be:

uint256 amount = address(this).balance;
feeRecipient.transfer(address(this).balance);
emit FeesSwept(address(0), amount, feeRecipient);

## Tools Used
Manual inspection

## Recommended Mitigation Steps
Store balance before calling transfer, as above.

