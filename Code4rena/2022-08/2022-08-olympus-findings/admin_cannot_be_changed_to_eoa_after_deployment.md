## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Admin cannot be changed to EOA after deployment](https://github.com/code-423n4/2022-08-olympus-findings/issues/94) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/main/src/Kernel.sol#L252-L253
https://github.com/code-423n4/2022-08-olympus/blob/main/src/modules/INSTR.sol#L52


# Vulnerability details

## Impact
After contracts are deployed and initialized, the admin address in `Kernel` contract can only be set to a contract. Granting and revoking roles will be possible to do only via a contract, which looks like an unintended behavior since these operations cannot be performed via governance (the governance contract is designed to be the only executor).


## Proof of Concept
Admin address can be changed to any address (EOA or contract) in the `executeAction` function in `Kernel`:
https://github.com/code-423n4/2022-08-olympus/blob/main/src/Kernel.sol#L252-L253

This piece explicitly allows EOA addresses since the other actions in the function (besides `ChangeExecutor`) are checked to have only a contract as the target (see `ensureContract` function calls in the other actions). This, and the fact that roles cannot be managed via governance, leads to the conclusion that an admin is designed to be an EOA.

However, in the `store` function in `INSTR`, action target can only be a contract:
https://github.com/code-423n4/2022-08-olympus/blob/main/src/modules/INSTR.sol#L52

After the contracts are deployed, `INSTR` will be the only contract that's allowed to call `Kernel.executeAction`:
https://github.com/code-423n4/2022-08-olympus/blob/main/src/scripts/Deploy.sol#L220

Thus, there will be no way to change admin to an EOA. If admin needs to be an EOA, the `INSTR` contract needs to be patched and re-deployed to allow non-contract targets.

## Tools Used

## Recommended Mitigation Steps
Allow EOA addresses as instruction targets or disallow non-contract admin addresses.