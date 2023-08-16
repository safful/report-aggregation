## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [`LimboDAO.killDAO()` doesn't update the DAO address of `FlanBackstop`, `UniswapHelper`, and `ProposalFactory`](https://github.com/code-423n4/2022-01-behodler-findings/issues/86) 

# Handle

Ruhum


# Vulnerability details

## Impact
`LimboDAO.killDAO()` is used to assign control of the contracts to a new DAO. Currently, it only updates `Flan` & `Limbo`. But, `FlanBackstop`, `UniswapHelper`, and `ProposalFactory` also depend on the DAO. Those are not updated. The new DAO loses control over them.

## Proof of Concept
https://github.com/code-423n4/2022-01-behodler/blob/main/contracts/DAO/LimboDAO.sol#L226

## Tools Used
none

## Recommended Mitigation Steps
Instead of calling `setDAO()` on hardcoded address, the function should allow passing an array of addresses for which `setDAO()` is called.

```sol
function killDAO(address[] calldata a, address newOwner) public onlyOwner isLive {
    domainConfig.live = false;
    for (uint i; i < a.length; i++) {
        Governable(a[i]).setDAO(newOwner);
    }
    emit daoKilled(newOwner);
  }
```

