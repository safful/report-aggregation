## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [No option to unlock funds before set duration](https://github.com/code-423n4/2022-01-xdefi-findings/issues/183) 

# Handle

sirhashalot


# Vulnerability details

## Impact

If a user locks funds in the contract, they can only withdraw funds by calling functions that in turn call the `_unlock()` function. The `_unlock()` function requires the position to have block.timestamp >= position.expiry. If there is a problem with the contract, with the XDEFI ERC20 token, or a user changes their mind and wants their funds back, they do not have this option. This can be more problematic with very long lock duration values.

## Proof of Concept

There is a hard requirement that block.timestamp >= uint256(expiry) for any position before it can be unlocked and the funds released. All code paths that allow a use to withdraw their XDEFI rely on the `_unlock()` function: 
https://github.com/XDeFi-tech/xdefi-distribution/blob/3856a42df295183b40c6eee89307308f196612fe/contracts/XDEFIDistribution.sol#L305

## Recommended Mitigation Steps

Different options exist to assist users with this issue. One would be to keep lock duration values small, especially when the contract is first released to users. Another is to add an emergency withdrawal function that has the onlyOwner modifier, such as using OpenZeppelin's Pausable module:
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/Pausable.sol 

