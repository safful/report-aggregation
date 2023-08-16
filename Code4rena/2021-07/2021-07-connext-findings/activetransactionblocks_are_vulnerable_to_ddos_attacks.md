## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [activeTransactionBlocks are vulnerable to DDoS attacks](https://github.com/code-423n4/2021-07-connext-findings/issues/27) 

# Handle

pauliax


# Vulnerability details

## Impact
There is a potential issue in function removeUserActiveBlocks and the for loop inside it. I assume you are aware of block gas limits (they may be less relevant on other chains but still needs to be accounted for), so as there is no limit for activeTransactionBlocks it may grow so large that the for loop may never finish. You should consider introducing an upper limit for activeTransactionBlocks. Also, a malicious actor may block any account (DDOS) by just calling prepare again and again with 0 amount acting as a router. This will push activeTransactionBlocks to the specified user until it is no longer possible to remove them from the array.

This is also a gas issue as function removeUserActiveBlocks iterating and assigning large dynamic arrays is very gas-consuming. Consider optimizing the algorithm, e.g. finding the first occurrence, then swap it with the last item, pop the array, and break. Or maybe even using an EnumerableMap so you can find and remove elements in O(1) https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/structs/EnumerableMap.sol
It depends on what is the usual number of activeTransactionBlocks. If it is expected to be low (e.g. less than 5), then the current approach will work, but with larger arrays, I expect EnumerableMap would be more efficient.

## Recommended Mitigation Steps
An upper limit will not fully mitigate this issue as a malicious actor can still DDOS the user by pushing useless txs until this limit is reached and a valid router may not be able to submit new txs. As you need to improve both the security and performance of removeUserActiveBlocks I think that EnumerableMap may be a go-to solution.

