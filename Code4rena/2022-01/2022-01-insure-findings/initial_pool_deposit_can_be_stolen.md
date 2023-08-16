## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Initial pool deposit can be stolen](https://github.com/code-423n4/2022-01-insure-findings/issues/250) 

# Handle

cmichel


# Vulnerability details

Note that the `PoolTemplate.initialize` function, called when creating a market with `Factory.createMarket`, calls a vault function to transfer an initial deposit amount (`conditions[1]`) _from_ the initial depositor (`_references[4]`):

```solidity
// PoolTemplate
function initialize(
     string calldata _metaData,
     uint256[] calldata _conditions,
     address[] calldata _references
) external override {
     // ...

     if (_conditions[1] > 0) {
          // @audit vault calls asset.transferFrom(_references[4], vault, _conditions[1])
          _depositFrom(_conditions[1], _references[4]);
     }
}

function _depositFrom(uint256 _amount, address _from)
     internal
     returns (uint256 _mintAmount)
{
     require(
          marketStatus == MarketStatus.Trading && paused == false,
          "ERROR: DEPOSIT_DISABLED"
     );
     require(_amount > 0, "ERROR: DEPOSIT_ZERO");

     _mintAmount = worth(_amount);
     // @audit vault calls asset.transferFrom(_from, vault, _amount)
     vault.addValue(_amount, _from, address(this));

     emit Deposit(_from, _amount, _mintAmount);

     //mint iToken
     _mint(_from, _mintAmount);
}
```

The initial depositor needs to first approve the vault contract for the `transferFrom` to succeed.

An attacker can then frontrun the `Factory.createMarket` transaction with their own market creation (it does not have access restrictions) and create a market _with different parameters_ but still passing in `_conditions[1]=amount` and `_references[4]=victim`.

A market with parameters that the initial depositor did not want (different underlying, old whitelisted registry/parameter contract, etc.) can be created with their tokens and these tokens are essentially lost.

## Recommended Mitigation Steps
Can the initial depositor be set to `Factory.createMarket`'s `msg.sender`, instead of being able to pick a whitelisted one as `_references[4]`?


