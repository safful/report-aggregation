## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed

# [Missing events for critical operations](https://github.com/code-423n4/2021-11-overlay-findings/issues/64) 

# Handle

WatchPug


# Vulnerability details

Across the contracts, there are certain critical operations that change critical values that affect the users of the protocol.

It's a best practice for these setter functions to emit events to record these changes on-chain for off-chain monitors/tools/interfaces to register the updates and react if necessary.

Instances include:

https://github.com/code-423n4/2021-11-overlay/blob/1833b792caf3eb8756b1ba5f50f9c2ce085e54d0/contracts/mothership/OverlayV1Mothership.sol#L75-L79

```solidity
function setOVL (address _ovl) external onlyGovernor {

        ovl = _ovl;

    }
```

https://github.com/code-423n4/2021-11-overlay/blob/1833b792caf3eb8756b1ba5f50f9c2ce085e54d0/contracts/mothership/OverlayV1Mothership.sol#L87-L116

```solidity
    function initializeMarket(address market) external onlyGovernor {

        require(!marketExists[market], "OVLV1:!!initialized");

        marketExists[market] = true;
        marketActive[market] = true;

        allMarkets.push(market);

    }

    /// @notice Disables an existing market contract for a mirin market
    function disableMarket(address market) external onlyGovernor {

        require(marketActive[market], "OVLV1: !enabled");

        marketActive[market] = false;

    }

    /// @notice Enables an existing market contract for a mirin market
    function enableMarket(address market) external onlyGovernor {

        require(marketExists[market], "OVLV1: !exists");

        require(!marketActive[market], "OVLV1: !disabled");

        marketActive[market] = true;

    }
```

And all functions in `OverlayV1Governance.sol`.

