## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [BaseRewardPool4626 is not IERC4626 compliant](https://github.com/code-423n4/2022-05-aura-findings/issues/26) 

# Lines of code

https://github.com/aurafinance/convex-platform/blob/9cae5eb5a77e73bbc1378ef213740c1889e2e8a3/contracts/contracts/BaseRewardPool4626.sol


# Vulnerability details

## Impact


BaseRewardPool4626 is not IERC4626 compliant.
This makes the BaseRewardPool4626 contract irrelevant as it is for now since projects won't be able to integrate with BaseRewardPool4626 using the[eip-4626](https://eips.ethereum.org/EIPS/eip-4626) standard.


## Suggestion

You can choose to remove the BaseRewardPool4626 and save on some deployment gas or review the necessary` functions` and `emits` required on [eip-4626](https://eips.ethereum.org/EIPS/eip-4626)  and add it to BaseRewardPool4626.


