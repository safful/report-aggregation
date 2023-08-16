## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [extra check setUnstakeWindow and setCooldown](https://github.com/code-423n4/2021-07-sherlock-findings/issues/18) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function setUnstakeWindow and setCooldown don't check that the input parameter isn't 0.
So the values could accidentally be set to 0 (although unlikely).
However you wouldn't want the to be 0 because that would allow attacks with flashloans (stake and unstake in the same transaction)

## Proof of Concept
https://github.com/code-423n4/2021-07-sherlock/blob/main/contracts/facets/Gov.sol#L124
 function setUnstakeWindow(uint40 _unstakeWindow) external override onlyGovMain {
    require(_unstakeWindow < 25000000, 'MAX'); // ~ approximate 10 years of blocks
    GovStorage.gs().unstakeWindow = _unstakeWindow;
  }

  function setCooldown(uint40 _period) external override onlyGovMain {
    require(_period < 25000000, 'MAX'); // ~ approximate 10 years of blocks
    GovStorage.gs().unstakeCooldown = _period;
  }

## Tools Used

## Recommended Mitigation Steps
Check the input parameter of setUnstakeWindow and setCooldown isn't 0




