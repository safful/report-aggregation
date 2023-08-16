## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- sponsor confirmed

# [`currentTemplateId` is Not Actively Removed by `MISOLauncher.removeLiquidityLauncherTemplate()`](https://github.com/code-423n4/2021-09-sushimiso-findings/issues/32) 

# Handle

leastwood


# Vulnerability details

## Impact

If the current template ID is removed from `MISOLauncher.sol`, the function `removeLiquidityLauncherTemplate()` does not accurately reflect this by deleting `currentTemplateId[_templateId]`. This may lead to users actively using a removed template, expecting the `deployLauncher()` function to succeed when it will revert instead.

## Proof of Concept

https://github.com/sushiswap/miso/blob/master/contracts/MISOLauncher.sol#L323-L334

## Tools Used

Manual code review

## Recommended Mitigation Steps

Consider removing `currentTemplateId[_templateId]` if the template to be removed by `removeLiquidityLauncherTemplate()` is the same template.

