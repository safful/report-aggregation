## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-04

# [Pool designed to be upgradeable but does not set owner, making it unupgradeable](https://github.com/code-423n4/2022-11-non-fungible-findings/issues/186) 

# Lines of code

https://github.com/code-423n4/2022-11-non-fungible/blob/323b7cbf607425dd81da96c0777c8b12e800305d/contracts/Pool.sol#L13


# Vulnerability details

## Description

The docs state:
"*The pool allows user to predeposit ETH so that it can be used when a seller takes their bid. It uses an ERC1967 proxy pattern and only the exchange contract is permitted to make transfers.*"

Pool is designed as an ERC1967 upgradeable proxy which handles balances of users in Not Fungible. Users may interact via deposit and withdraw with the pool, and use the funds in it to pay for orders in the Exchange.

Pool is declared like so:
```
contract Pool is IPool, OwnableUpgradeable, UUPSUpgradeable {
	function _authorizeUpgrade(address) internal override onlyOwner {}
	...
```

Importantly, it has no constructor and no initializers. The issue is that when using upgradeable contracts, it is important to implement an initializer which will call the base contract's initializers in turn. See how this is done correctly in Exchange.sol:

```
/* Constructor (for ERC1967) */
function initialize(
    IExecutionDelegate _executionDelegate,
    IPolicyManager _policyManager,
    address _oracle,
    uint _blockRange
) external initializer {
    __Ownable_init();
    isOpen = 1;
	  ...
}
```

Since Pool skips the \_\_Ownable_init initialization call, this logic is skipped:
```
function __Ownable_init() internal onlyInitializing {
    __Ownable_init_unchained();
}
function __Ownable_init_unchained() internal onlyInitializing {
    _transferOwnership(_msgSender());
}
```

Therefore, the contract owner stays zero initialized, and this means any use of onlyOwner will always revert.

The only use of onlyOwner in Pool is here:
```
function _authorizeUpgrade(address) internal override onlyOwner {}
```

The impact is that when the upgrade mechanism will check caller is authorized, it will revert. Therefore, the contract is unexpectedly unupgradeable. Whenever the EXCHANGE or SWAP address, or some functionality needs to be changed, it would not be possible.

## Impact

The Pool contract is designed to be upgradeable but is actually not upgradeable

## Proof of Concept

In the 'pool' test in execution.test.ts, add the following lines:
```
it('owner configured correctly', async () => {
  expect(await pool.owner()).to.be.equal(admin.address);
});
```

It shows that the pool after deployment has owner as 0x0000...00

## Tools Used

Manual audit, hardhat

## Recommended Mitigation Steps

Implement an initializer for Pool similarly to the Exchange.sol contract.