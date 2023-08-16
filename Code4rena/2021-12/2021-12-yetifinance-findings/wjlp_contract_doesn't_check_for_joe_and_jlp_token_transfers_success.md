## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [WJLP contract doesn't check for JOE and JLP token transfers success](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/107) 

# Handle

hyh


# Vulnerability details

# Impact

Transactions will not be reverted on failed transfer call, setting system state as if it was successful.
This will lead to wrong state accounting down the road with a wide spectrum of possible consequences.

## Proof of Concept

_safeJoeTransfer do not check for JOE.transfer call success:
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/AssetWrappers/WJLP/WJLP.sol#L268

_safeJoeTransfer is called by _sendJoeReward, which is used in reward claiming.

JOE token use transfer from OpenZeppelin ERC20:
https://github.com/traderjoe-xyz/joe-core/blob/main/contracts/JoeToken.sol#L9

Which does return success code:
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol#L113

Trader Joe also uses checked transfer when dealing with JOE tokens:
https://github.com/traderjoe-xyz/joe-core/blob/main/contracts/MasterChefJoeV3.sol#L102


Also, unwrapFor do not check for JLP.transfer call success:
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/AssetWrappers/WJLP/WJLP.sol#L166


## Recommended Mitigation Steps

Add a require() check for the success of JOE transfer in _safeJoeTransfer function and create and use a similar function with the same check for JLP token transfers

