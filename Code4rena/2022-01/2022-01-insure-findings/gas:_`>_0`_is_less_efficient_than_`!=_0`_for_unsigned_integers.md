## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: `> 0` is less efficient than `!= 0` for unsigned integers](https://github.com/code-423n4/2022-01-insure-findings/issues/36) 

# Handle

Dravee


# Vulnerability details

## Impact  
`!= 0` costs less gas compared to `> 0` for unsigned integer  
  
## Proof of Concept  
`> 0` is used in the following location(s):
```  
CDSTemplate.sol:100:                bytes(_metaData).length > 0 &&
CDSTemplate.sol:132:        require(_amount > 0, "ERROR: DEPOSIT_ZERO");
CDSTemplate.sol:140:        if (_supply > 0 && _liquidity > 0) {
CDSTemplate.sol:142:        } else if (_supply > 0 && _liquidity == 0) {
CDSTemplate.sol:191:        require(_amount > 0, "ERROR: REQUEST_ZERO");
CDSTemplate.sol:223:        require(_amount > 0, "ERROR: WITHDRAWAL_ZERO");
CDSTemplate.sol:296:        if (totalSupply() > 0) {
Factory.sol:175:        if (_references.length > 0) {
Factory.sol:185:        if (_conditions.length > 0) {
Factory.sol:187:                if (conditionlist[address(_template)][i] > 0) {
IndexTemplate.sol:133:                bytes(_metaData).length > 0 &&
IndexTemplate.sol:166:        require(_amount > 0, "ERROR: DEPOSIT_ZERO");
IndexTemplate.sol:172:        if (_supply > 0 && _totalLiquidity > 0) {
IndexTemplate.sol:174:        } else if (_supply > 0 && _totalLiquidity == 0) {
IndexTemplate.sol:199:        require(_amount > 0, "ERROR: REQUEST_ZERO");
IndexTemplate.sol:231:        require(_amount > 0, "ERROR: WITHDRAWAL_ZERO");
IndexTemplate.sol:246:        if (_liquidityAfter > 0) {
IndexTemplate.sol:274:        if(_totalLiquidity > 0){
IndexTemplate.sol:283:                if (_allocPoint > 0) {
IndexTemplate.sol:391:                if (_current > _target && _available != 0) {
IndexTemplate.sol:427:            allocPoints[msg.sender] > 0,
IndexTemplate.sol:477:        require(allocPoints[msg.sender] > 0);
IndexTemplate.sol:493:        if (totalLiquidity() > 0) {
IndexTemplate.sol:513:        if (totalSupply() > 0) {
IndexTemplate.sol:612:        if (totalAllocPoint > 0) {
IndexTemplate.sol:656:            if (allocPoints[poolList[i]] > 0) {
InsureDAOERC20.sol:302:        require(accountBalance >= amount, "ERC20: burn amount exceeds balance");
Parameters.sol:31:    mapping(address => uint256) private _fee; //fee rate in 1e6 (100% = 1e6)
PoolTemplate.sol:185:                bytes(_metaData).length > 0 &&
PoolTemplate.sol:218:        if (_conditions[1] > 0) {
PoolTemplate.sol:237:        require(_amount > 0, "ERROR: DEPOSIT_ZERO");
PoolTemplate.sol:263:        require(_amount > 0, "ERROR: DEPOSIT_ZERO");
PoolTemplate.sol:282:        require(_amount > 0, "ERROR: REQUEST_ZERO");
PoolTemplate.sol:321:        require(_amount > 0, "ERROR: WITHDRAWAL_ZERO");
PoolTemplate.sol:391:        } else if (_index.credit > 0) {
PoolTemplate.sol:396:            if (_pending > 0) {
PoolTemplate.sol:401:        if (_credit > 0) {
PoolTemplate.sol:437:        if (_credit > 0) {
PoolTemplate.sol:444:        if (_pending > 0) {
PoolTemplate.sol:521:        if (_totalCredit > 0) {
PoolTemplate.sol:672:            if (indicies[indexList[i]].credit > 0) {
PoolTemplate.sol:706:            if (_credit > 0) {
PoolTemplate.sol:726:        if (_deductionFromPool > 0) {
PoolTemplate.sol:745:        if (totalSupply() > 0) {
PoolTemplate.sol:802:        if (_supply > 0 && _originalLiquidity > 0) {
PoolTemplate.sol:804:        } else if (_supply > 0 && _originalLiquidity == 0) {
PoolTemplate.sol:835:        if (totalLiquidity() > 0) {
PoolTemplate.sol:847:        if (lockedAmount > 0) {
PoolTemplate.sol:929:        require(b > 0);
Vault.sol:154:            attributions[msg.sender] > 0 &&
Vault.sol:187:            attributions[msg.sender] > 0 &&
Vault.sol:220:            attributions[msg.sender] > 0 &&
Vault.sol:347:        if (_amount > 0) {
Vault.sol:388:        if (totalAttributions > 0 && _attribution > 0) {
Vault.sol:406:        if (attributions[_target] > 0) {
Vault.sol:473:        } else if (IERC20(_token).balanceOf(address(this)) > 0) {
```  
  
## Tools Used  
VS Code  
  
## Recommended Mitigation Steps  
Change `> 0` with `!= 0`.


