## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [No max for advanceIncentive](https://github.com/code-423n4/2021-11-malt-findings/issues/190) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function setAdvanceIncentive of DAO.sol doesn't check for a maximum value of incentive.
If incentivewould be very large, then advanceIncentive would be very large and the function advance() would mint a large amount of malt.

The function setAdvanceIncentive() can only be called by an admin, but a mistake could be made.
Also if an admin would want to do a rug pull, this would be an ideal place to do it.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/DAO.sol#L98-L104

```JS
  function setAdvanceIncentive(uint256 incentive)  externalonlyRole(ADMIN_ROLE, "Must have admin role") {
   ...
    advanceIncentive = incentive;
```

https://github.com/code-423n4/2021-11-malt/blob/d3f6a57ba6694b47389b16d9d0a36a956c5e6a94/src/contracts/DAO.sol#L55-L63

```JS
function advance() external {
...
    malt.mint(msg.sender, advanceIncentive * 1e18);

```

## Tools Used

## Recommended Mitigation Steps
Check for a reasonable maximum value in advance()

