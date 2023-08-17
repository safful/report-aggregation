## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed
- valid

# [QA Report](https://github.com/code-423n4/2022-06-badger-findings/issues/127) 

### Condition in `receive` function can be bypassed with self-destruct of another contract

**Details**: The logic in [L436-L438](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L436-L438) implies that the contract should only receive ether if `isClaimingBribes` is `true`. However, this check can be bypassed by deploying a contract (say, Attacker) and setting up the address of MyStrategy contract as the destination of a `selfdestruct` in the Attacker contract — for more information and otherway to bypass the require of [L437](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L437), see [this link](https://docs.soliditylang.org/en/latest/contracts.html#receive-ether-function).

**Impact**: Informational (could possibly break internal calculations of the protocol though)

### **Re-entrancy guard upgradeable contract is not initialized**

**Details**: As stated in OpenZeppelin [docs](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#:~:text=When%20writing%20an%20initializer%2C%20you%20need%20to%20take%20special%20care%20to%20manually%20call%20the%20initializers%20of%20all%20parent%20contracts), “when writing an initializer, you need to take special care to manually call the initializers of all parent contracts”. However, the initializer of `ReentrancyGuardUpgradeable` is not called.

**Mitigation**: Ensure that all necessary functions are inherited from the upgradeable contracts.

**Impact**: Code QA

### TODOs are left in comments

**Details**: In [L284](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L284) and [L422](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L422) of ****MyStrategy.sol**** there are comments with TODOs. These should be resolved and removed from the code before deployment.

**Mitigation**: Check the TODOs and fix/remove them.

**Impact**: Code QA

### Usage of deprecated function safeApprove

**Details**: In [L65-68](https://github.com/Badger-Finance/vested-aura/blob/d504684e4f9b56660a9e6c6dfb839dcebac3c174/contracts/MyStrategy.sol#L65-L68) of ****MyStrategy.sol**** the function `safeApprove` from OpenZeppelin contracts are used, however these functions have been deprecated according to [OpenZeppelin 3.x docs](https://docs.openzeppelin.com/contracts/3.x/api/token/erc20#SafeERC20-safeApprove-contract-IERC20-address-uint256-) (note that ****MyStrategy.sol**** correctly use OpenZeppelin 3.4.0).

**Impact**: Code QA

### Alert developers that OpenZeppelin contract cannot be bumped to 4.x

**Note to judges**: I think this issue is out-of-scope, but worthy to inform anyway 🙂

**Details**: According to [brownie config file](https://github.com/Badger-Finance/vested-aura/blob/v0.0.2/brownie-config.yaml), the contract **MyStrategy.sol** imports version 3.4.0 of OpenZeppelin’s SafeMath, and this is the recommend version to use with Solidity 0.6.12.

Unware developers, however, may want to bump OpenZeppelin version to the lastest one, and running `brownie compile` will compile the contract without errors (at least for 4.6.0). However, as alerted by the comments in [L6-8](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/51e11611c40ec1ad772e2a075cdc8487bbadf8ad/contracts/utils/math/SafeMathUpgradeable.sol#L6-L8), recent versions of SafeMath *should only be used with Solidity 0.8 or later, because it relies on the compiler's built in overflow checks*. This implies that checks of overflow/underflow will not be used, and this could be further exploited in other attacks. 

This could also be particularly dangerous in the scenario wherein a developer does this bumping while writing his own MyStrategy contract, since he will probably use the SafeMath functions assuming that underflow/overflow checks are being used in his code.

**Mitigation**: Consider adding a comment in brownie config file alerting the users that OpenZeppelin version should not be bumped.

**Impact**: Informational (probably out-of-scope)