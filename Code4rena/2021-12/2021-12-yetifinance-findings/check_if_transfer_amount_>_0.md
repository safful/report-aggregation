## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Check if transfer amount > 0](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/171) 

# Handle

Dravee


# Vulnerability details

## Impact
Checking non-zero transfer values can avoid an external call to save gas. 

## Proof of Concept
Instances missing a non-zero check:
```
ActivePool.sol:
  156:         bool sent = IERC20(_collateral).transfer(_to, _amount);

BorrowerOperations.sol:
  742:             bool transferredToActivePool = coll.transferFrom(_from, address(activePool), amount);

DefaultPool.sol:
  121:         bool success = IERC20(_collateral).transfer(activePool, _amount);

StabilityPool.sol:
  947:                 IERC20(assets[i]).transfer(_to, amounts[i]);

TeamAllocation.sol:
  69:             require(YETI.transfer(member, amount));
  77:         YETI.transfer(_to, _amount);

YetiFinanceTreasury.sol:
  25:         _token.transfer(_to, _amount);

AssetWrappers\WJLP\WJLP.sol:
  127:         JLP.transferFrom(_from, address(this), _amount);
  166:         JLP.transfer(_to, _amount);
  273:             JOE.transfer(_to, _amount);

Dependencies\LiquityBase.sol:
  170:             if (!token.transfer(_to, _coll.amounts[i])) {

LPRewards\Dependencies\SafeERC20.sol:
  23:         _callOptionalReturn(token, abi.encodeWithSelector(token.transfer.selector, to, value));
  27:         _callOptionalReturn(token, abi.encodeWithSelector(token.transferFrom.selector, from, to, value));

YETI\CommunityIssuance.sol:
  125:         yetiToken.transfer(_account, _YETIamount);

YETI\LockupContract.sol:
  68:         yetiTokenCached.transfer(beneficiary, YETIBalance);

YETI\ShortLockupContract.sol:
  67:         yetiTokenCached.transfer(beneficiary, YETIBalance);

YETI\sYETIToken.sol:
  203:         yetiToken.transfer(to, amount);

YETI\TeamLockup.sol:
  47:             require(YETI.transfer(multisig, _amount));
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Check if transfer amount > 0.
It is done at some places already, like here: https://github.com/code-423n4/2021-12-yetifinance/blob/1da782328ce4067f9654c3594a34014b0329130a/packages/contracts/contracts/LPRewards/Unipool.sol#L189-L192

