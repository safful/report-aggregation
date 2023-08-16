## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Migrate old balance on setToken](https://github.com/code-423n4/2022-01-livepeer-findings/issues/234) 

# Handle

pauliax


# Vulnerability details

## Impact
In contract BridgeMinter function setToken, it just sets the new tokenAddr, but it does not process the old token balance leaving it stuck in the contract. I think that setToken could also migrate the old balance somewhere before updating the token address.
I can even suggest adding token rescue functions to the contracts that may come in handy in such cases or if someone accidentally sends the tokens directly to the contract. An owner can rescue the tokens if the token is not protected (e.g. intended to be held in the contract).

## Recommended Mitigation Steps
An example implementation that could help to rescue old token balance:
```solidity
  function withdrawLPTToL1Migrator(address _tokenAddr, address _recipient) external onlyControllerOwner returns (uint256) {
      require(_tokenAddr != tokenAddr, "protected");

      IERC20 token = IERC20(_tokenAddr);

      uint256 balance = token.balanceOf(address(this));

      token.transfer(_recipient, balance);

      return balance;
  }
```

