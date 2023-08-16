## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Improve precision and gas costs in_shareMochi](https://github.com/code-423n4/2021-10-mochi-findings/issues/150) 

# Handle

pauliax


# Vulnerability details

## Impact
This not only loses some precision (cuz of multiplication and division) but also consumes more gas:
    // send Mochi to vMochi Vault
    mochi.transfer(
        address(engine.vMochi()),
        (mochiBalance * vMochiRatio) / 1e18
    );
    // send Mochi to veCRV Holders
    mochi.transfer(
        crvVoterRewardPool,
        (mochiBalance * (1e18 - vMochiRatio)) / 1e18
    );

## Recommended Mitigation Steps
Proposed improvement:
  // send Mochi to vMochi Vault
  uint toVault = (mochiBalance * vMochiRatio) / 1e18;
  mochi.transfer(
      address(engine.vMochi()),
      toVault
  );
  // send Mochi to veCRV Holders
  mochi.transfer(
      crvVoterRewardPool,
      mochiBalance - toVault;
  );

This way you the whole mochiBalance will be transferred and it will cost less to do that as fewer math operations are performed.

