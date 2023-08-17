## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [removeWrapping can be called when there are still wrapped tokens](https://github.com/code-423n4/2022-07-axelar-findings/issues/23) 

# Lines of code

https://github.com/code-423n4/2022-07-axelar/blob/a1205d2ba78e0db583d136f8563e8097860a110f/xc20/contracts/XC20Wrapper.sol#L66


# Vulnerability details

## Impact
An owner can call `removeWrapping`, even if there are still circulating wrapped tokens. This will cause the unwrapping of those tokens to fail, as `unwrapped[wrappedToken]` will be `addres(0)`.

## Recommended Mitigation Steps
Track how many wrapped tokens are in circulation, only allow the removal of a wrapped tokens when there are 0 to ensure for users that they will always be able to unwrap.