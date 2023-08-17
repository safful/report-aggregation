## Tags

- bug
- 2 (Med Risk)
- primary issue
- sponsor confirmed
- edited-by-warden
- selected for report
- M-01

# [diamondCut is not protected in case of governor's key leakage](https://github.com/code-423n4/2022-10-zksync-findings/issues/46) 

# Lines of code

https://github.com/code-423n4/2022-10-zksync/blob/4db6c596931a291b17a4e0e2929adf810a4a0eed/ethereum/contracts/zksync/facets/DiamondCut.sol#L46
https://github.com/code-423n4/2022-10-zksync/blob/4db6c596931a291b17a4e0e2929adf810a4a0eed/ethereum/contracts/zksync/libraries/Diamond.sol#L277


# Vulnerability details

## Impact

When the governor proposes a diamondCut, governor must wait for `upgradeNoticePeriod` to be passed, or security council members have to approve the proposal to bypass the notice period, so that the governor can execute the proposal.

```
   require(approvedBySecurityCouncil || upgradeNoticePeriodPassed, "a6"); // notice period should expire
   require(approvedBySecurityCouncil || !diamondStorage.isFrozen, "f3");
```

If the governor's key is leaked and noticed by zkSync, the attacker must wait for the notice period to execute the already proposed diamondCut with the malicious `_calldata` based on the note below from zkSync, or to propose a new malicious diamondCut. For, both cases, the attacker loses time.

>NOTE: proposeDiamondCut - commits data associated with an upgrade but does not execute it. While the upgrade is associated with facetCuts and (address _initAddress, bytes _calldata) the upgrade will be committed to the facetCuts and _initAddress. This is done on purpose, to leave some freedom to the governor to change calldata for the upgrade between proposing and executing it.

Since, there is a notice period (as zkSync noticed the key leakage, security council member will not approve the proposal, so bypassing the notice period is not possible), there is enough time for zkSync to apply security measures (pausing any deposit/withdraw, reporting in media to not execute any transaction in zkSync, and so on).

But, the attacker can be smarter, just before the proposal be executed by the governor (i.e. the notice period is passed or security council members approved it), the attacker executes the proposal earlier than governor with the malicious `_calldata`. In other words, the attacker front runs the governor.

Therefore, if zkSync notices the governor's key leakage beforehand, there is enough time to protect the project. But, if zkSync does not notice the governor's key leakage, the attacker can change the `_calldata` into a malicious one in the last moment so that it is not possible to protect the project.


## Proof of Concept
https://github.com/code-423n4/2022-10-zksync/blob/4db6c596931a291b17a4e0e2929adf810a4a0eed/ethereum/contracts/zksync/libraries/Diamond.sol#L277
https://github.com/code-423n4/2022-10-zksync/blob/4db6c596931a291b17a4e0e2929adf810a4a0eed/ethereum/contracts/zksync/facets/DiamondCut.sol#L46


## Tools Used

## Recommended Mitigation Steps

`_calldata` should be included in the proposed diamondCut:
https://github.com/code-423n4/2022-10-zksync/blob/4db6c596931a291b17a4e0e2929adf810a4a0eed/ethereum/contracts/zksync/facets/DiamondCut.sol#L27

Or, at least one of the security council members should approve the `_calldata` during execution of the proposal.