## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Cache storage variables in the stack can save gas](https://github.com/code-423n4/2021-10-covalent-findings/issues/53) 

# Handle

WatchPug


# Vulnerability details

For the storage variables that will be accessed multiple times, cache them in the stack can save ~100 gas from each extra read (`SLOAD` after Berlin).

For example:

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L183-L188

```solidity
if (msg.sender == v._address){
    require(amount + v.stakings[msg.sender].staked >= validatorMinStakedRequired, "Amount is less than minimum staked required");
}
else {
    // otherwise need to check for max cap
    uint128 validatorStaked = v.stakings[v._address].staked;
```

`v._address` is read twice.

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L345-L351

```solidity
function addValidator(address validator, address operator, uint128 commissionRate) public onlyOwner {
        validators[validatorsN]._address = validator;
        validators[validatorsN].operator = operator;
        validators[validatorsN].commissionRate = commissionRate;
        emit ValidatorAdded(validatorsN, validator, operator);
        validatorsN +=1;
    }
```

`validatorsN` is read 4 times.

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L437-L446

```solidity
function getValidatorsDetails() public view returns (uint128[] memory commissionRates, uint128[] memory delegated) {
        commissionRates = new uint128[](validatorsN);
        delegated = new uint128[](validatorsN);
        for (uint128 i = 0; i < validatorsN; i++){
            Validator storage v = validators[i];
            commissionRates[i] = v.commissionRate;
            delegated[i] = v.delegated - v.stakings[v._address].staked;
        }
        return (commissionRates, delegated);
    }
```

`validatorsN` is read 3 times.

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L450-L459

```solidity
function getDelegatorDetails(address delegator) public view returns( uint[] memory delegated,  uint[] memory rewardsAvailable, uint[] memory commissionRewards) {
       delegated = new uint[](validatorsN);
       rewardsAvailable = new uint[](validatorsN);
       commissionRewards = new uint[](validatorsN);
       uint256 currentEpoch = block.number < endEpoch? block.number: endEpoch;
       uint128 newGlobalExchangeRate = uint128((uint256(allocatedTokensPerEpoch) * divider/totalGlobalShares)*(currentEpoch - lastUpdateEpoch)) + globalExchangeRate;

        for (uint128 i = 0; i < validatorsN; i++){
            Validator storage v = validators[i];

```

`validatorsN` is read 4 times.

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L313-L322

```solidity
if(msg.sender == v._address){
    require(rewards + v.commissionAvailableToRedeem >= amount, "Cannot redeem more than available");
    // first redeem rewards from commission
    uint128 commissionLeftOver = amount < v.commissionAvailableToRedeem ? v.commissionAvailableToRedeem - amount : 0;
    // if there is more, redeem  it from regular rewards
    if (commissionLeftOver == 0){
        uint128 validatorSharesRemove = tokensToShares(amount - v.commissionAvailableToRedeem, v.exchangeRate);
        s.shares -= validatorSharesRemove;
        v.totalShares -= validatorSharesRemove;
    }
```

`v.commissionAvailableToRedeem` is read 4 times.

