## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [DOS by Frontrunning NoteERC20 `initialize()` Function](https://github.com/code-423n4/2021-08-notional-findings/issues/7) 

# Handle

leastwood


# Vulnerability details

## Impact

The `scripts/` folder outlines a number of deployment scripts used by the Notional team. Some of the contracts deployed utilise the ERC1967 upgradeable proxy standard. This standard involves first deploying an implementation contract and later a proxy contract which uses the implementation contract as its logic. When users make calls to the proxy contract, the proxy contract will delegate call to the underlying implementation contract. `NoteERC20.sol` and `Router.sol` both implement an `initialize()` function which aims to replace the role of the `constructor()` when deploying proxy contracts. It is important that these proxy contracts are deployed and initialized in the same transaction to avoid any malicious frontrunning. However, `scripts/deployment.py` does not follow this pattern when deploying `NoteERC20.sol`'s proxy contract. As a result, a malicious attacker could monitor the Ethereum blockchain for bytecode that matches the `NoteERC20` contract and frontrun the `initialize()` transaction to gain ownership of the contract. This can be repeated as a Denial Of Service (DOS) type of attack, effectively preventing Notional's contract deployment, leading to unrecoverable gas expenses.

## Proof of Concept

https://github.com/code-423n4/2021-08-notional/blob/main/scripts/deployment.py#L44-L60
https://github.com/code-423n4/2021-08-notional/blob/main/scripts/mainnet/deploy_governance.py#L71-L105

## Tools Used

Manual code review

## Recommended Mitigation Steps

As the `GovernanceAlpha.sol` and `NoteERC20.sol` are co-dependent contracts in terms of deployment, it won't be possible to deploy the governance contract before deploying and initializing the token contract. Therefore, it would be worthwhile to ensure the `NoteERC20.sol` proxy contract is deployed and initialized in the same transaction, or ensure the `initialize()` function is callable only by the deployer of the `NoteERC20.sol` contract. This could be set in the proxy contracts `constructor()`. 

