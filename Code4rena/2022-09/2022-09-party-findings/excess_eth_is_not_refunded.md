## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Excess eth is not refunded](https://github.com/code-423n4/2022-09-party-findings/issues/186) 

# Lines of code

https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/proposals/ArbitraryCallsProposal.sol#L72


# Vulnerability details

## Impact
The ArbitraryCallsProposal contract requires sender to provide eth(msg.value) for each call. Now if user has provided more eth than combined call.value then this excess eth is not refunded back to user

## Proof of Concept

1. Observe the [_executeArbitraryCalls function](https://github.com/PartyDAO/party-contracts-c4/blob/main/contracts/proposals/ArbitraryCallsProposal.sol#L37)

```
function _executeArbitraryCalls(
        IProposalExecutionEngine.ExecuteProposalParams memory params
    )
        internal
        returns (bytes memory nextProgressData)
    {

...
uint256 ethAvailable = msg.value;
        for (uint256 i = 0; i < calls.length; ++i) {
            // Execute an arbitrary call.
            _executeSingleArbitraryCall(
                i,
                calls[i],
                params.preciousTokens,
                params.preciousTokenIds,
                isUnanimous,
                ethAvailable
            );
            // Update the amount of ETH available for the subsequent calls.
            ethAvailable -= calls[i].value;
            emit ArbitraryCallExecuted(params.proposalId, i, calls.length);
        }
....
}
```

2. As we can see user provided msg.value is deducted with each calls[i].value
3. Assume user provided 5 amount as msg.value and made a single call with calls[0].value as 4
4. This means after calls have been completed ethAvailable will become 5-4=1
5. Ideally this 1 eth should be refunded back to user but there is no provision for same and the fund will remain in contract


## Recommended Mitigation Steps
At the end of _executeArbitraryCalls function, refund the remaining ethAvailable back to the user