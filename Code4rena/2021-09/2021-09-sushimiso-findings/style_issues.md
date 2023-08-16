## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Style issues](https://github.com/code-423n4/2021-09-sushimiso-findings/issues/130) 

# Handle

pauliax


# Vulnerability details

## Impact
Style issues that you may want to apply or reject, no impact on security. Grouping them together as one submission to reduce waste. Consider fixing or ignoring them, up to you.

* Misleading comment here (similarly with launcherInfo):
  /// @notice Mapping from auction address created through this contract to Auction struct.
  mapping(address => Token) public tokenInfo;

* In function deployToken this check should have an error message to indicate the user what's wrong:
  require(tokenTemplates[_templateId] != address(0));
something like "MISOTokenFactory: incorrect _templateId". Also a meaningful revert message is missing here as regular user may not understand that token2 decimals should be >= token1 decimals:
  require(d2 >= d1);

* There are a few copypasted misleading error messages, e.g. in function setCurrentTemplateId:
     require(tokenTemplates[_templateId] != address(0), "MISOMarket: incorrect _templateId");
     require(IMisoToken(tokenTemplates[_templateId]).tokenTemplate() == _templateType, "MISOMarket: incorrect _templateType");
should be MISOTokenFactory, not MISOMarket. Here also indicates the wrong location:
   require(templateType > 0, "MISOLauncher: Incorrect template code ");
You should consider revisiting and fixing them.

* There are hardcoded magic numbers, e.g. in MISOTokenFactory 1000 is indicating 100%. It would make code more readable and maintainable if you extract such numbers as constants.

* There is so much duplicated code across Auction contracts. Consider introducing an abstract BaseAuction (or similar name) contract that has common functions that specific auctions can inherit, e.g. ETH_ADDRESS, marketParticipationAgreement, revertBecauseUserDidNotProvideAgreement, etc.



