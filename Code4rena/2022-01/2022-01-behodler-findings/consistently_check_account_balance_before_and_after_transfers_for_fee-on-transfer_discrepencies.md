## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Consistently check account balance before and after transfers for Fee-On-Transfer discrepencies](https://github.com/code-423n4/2022-01-behodler-findings/issues/237) 

# Handle

Dravee


# Vulnerability details

## Impact
Wrong fateBalance bookkeeping for a user.
Wrong fateCreated value emitted.

## Proof of Concept
Taking into account the FOT is done almost everywhere important in the solution already. That's a known practice in the solution.

However, it's missing here (see @audit-info tags):
```
File: LimboDAO.sol
383:   function burnAsset(address asset, uint256 amount) public isLive incrementFate {
384:     require(assetApproved[asset], "LimboDAO: illegal asset");
385:     address sender = _msgSender();
386:     require(ERC677(asset).transferFrom(sender, address(this), amount), "LimboDAO: transferFailed"); //@audit-info FOT not taken into account
387:     uint256 fateCreated = fateState[_msgSender()].fateBalance;
388:     if (asset == domainConfig.eye) {
389:       fateCreated = amount * 10; //@audit-info wrong amount due to lack of FOT calculation
390:       ERC677(domainConfig.eye).burn(amount);//@audit-info wrong amount due to lack of FOT calculation
391:     } else {
392:       uint256 actualEyeBalance = IERC20(domainConfig.eye).balanceOf(asset);
393:       require(actualEyeBalance > 0, "LimboDAO: No EYE");
394:       uint256 totalSupply = IERC20(asset).totalSupply();
395:       uint256 eyePerUnit = (actualEyeBalance * ONE) / totalSupply;
396:       uint256 impliedEye = (eyePerUnit * amount) / ONE;//@audit-info wrong amount due to lack of FOT calculation
397:       fateCreated = impliedEye * 20;
398:     }
399:     fateState[_msgSender()].fateBalance += fateCreated; //@audit-info potentially wrong fateCreated as fateCreated can be equal to amount * 10;  
400:     emit assetBurnt(_msgSender(), asset, fateCreated);//@audit-info potentially wrong fateCreated emitted
401:   }
```

## Tools Used
VS Code

## Recommended Mitigation Steps
Check the balance before and after the transfer to take into account the Fees-On-Transfer

