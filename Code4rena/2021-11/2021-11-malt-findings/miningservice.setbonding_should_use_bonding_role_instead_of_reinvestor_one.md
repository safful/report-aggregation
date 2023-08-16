## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [MiningService.setBonding should use BONDING role instead of REINVESTOR one](https://github.com/code-423n4/2021-11-malt-findings/issues/370) 

# Handle

hyh


# Vulnerability details

## Impact

BONDING_ROLE cannot be managed after it was initialized.

## Proof of Concept

```setBonding``` set the wrong role via _swapRole:

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/MiningService.sol#L116

## Recommended Mitigation Steps

Set ```BONDING_ROLE``` instead of ```REINVESTOR_ROLE``` in ```setBonding``` function:

Now:
```
function setBonding(address _bonding)
	public
	onlyRole(ADMIN_ROLE, "Must have admin privs")
{
	require(_bonding != address(0), "Cannot use address 0");
	_swapRole(_bonding, bonding, REINVESTOR_ROLE);
	bonding = _bonding;
}
```

To be:
```
function setBonding(address _bonding)
	public
	onlyRole(ADMIN_ROLE, "Must have admin privs")
{
	require(_bonding != address(0), "Cannot use address 0");
	_swapRole(_bonding, bonding, BONDING_ROLE);
	bonding = _bonding;
}
```

