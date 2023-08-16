## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Potential DOS in Contracts Inheriting `UUPSUpgradeable.sol`](https://github.com/code-423n4/2021-08-notional-findings/issues/98) 

# Handle

leastwood


# Vulnerability details

## Impact

There are a number of contracts which inherit `UUPSUpgradeable.sol`, namely; `GovernanceAction.sol`, `PauseRouter.sol` and `NoteERC20.sol`. All these contracts are deployed using a proxy pattern whereby the implementation contract is used by the proxy contract for all its logic. The proxy contract will make delegate calls to the implementation contract. This helps to facilitate future upgrades by pointing the proxy contract to a new and upgraded implementation contract. However, if the implementation contract is left uninitialized, it is possible for any user to gain ownership of the `onlyOwner` role in the implementation contract for `NoteERC20.sol`. Once the user has ownership they are able to perform an upgrade of the implementation contract's logic contract and delegate call into any arbitrary contract, allowing them to self-destruct the proxy's implementation contract. Consequently, this will prevent all `NoteERC20.sol` interactions until a new implementation contract is deployed.

## Proof of Concept

Initial information about this issue was found [here](https://forum.openzeppelin.com/t/security-advisory-initialize-uups-implementation-contracts/15301).

Consider the following scenario:
- Notional finance deploys their contracts using their deployment scripts. These deployment scripts leave the implementation contracts uninitialized. Specifically the contract in question is `NoteERC20.sol`.
- This allows any arbitrary user to call `initialize()` on the `NoteERC20.sol` implementation contract.
- Once a user has gained control over `NoteERC20.sol`'s implementation contract, they can bypass the `_authorizeUpgrade` check used to restrict upgrades to the `onlyOwner` role.
- The malicious user then calls `UUPSUpgradeable.upgradeToAndCall()` shown [here](https://github.com/code-423n4/2021-08-notional/blob/main/contracts/proxy/utils/UUPSUpgradeable.sol#L40-L43) which in turn calls [this](https://github.com/code-423n4/2021-08-notional/blob/main/contracts/proxy/ERC1967/ERC1967Upgrade.sol#L77-L107) function. The new implementation contract then points to their own contract containing a self-destruct call in its fallback function.
- As a result, the implementation contract will be self-destructed due the user controlled delegate call shown [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v3.4.0-solc-0.7/contracts/utils/Address.sol#L163-L169), preventing all future calls to the `NoteERC20.sol` proxy contract until a new implementation contract has been deployed.

## Tools Used

Manual code review

## Recommended Mitigation Steps

Consider initializing the implementation contract for `NoteERC20.sol` and checking the correct permissions before deploying the proxy contract or performing any contract upgrades. This will help to ensure the implementation contract cannot be self-destructed.

