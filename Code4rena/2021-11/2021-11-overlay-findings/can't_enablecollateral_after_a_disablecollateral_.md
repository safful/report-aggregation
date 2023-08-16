## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Can't enableCollateral after a disableCollateral ](https://github.com/code-423n4/2021-11-overlay-findings/issues/55) 

# Handle

gpersoon


# Vulnerability details

## Impact
The function disableCollateral of OverlayV1Mothership.sol doesn't set collateralActive[_collateral] = false;
But it does revoke the roles.

Now enableCollateral  can never be used because collateralActive[_collateral] ==true  and it will never pass the second require.
So you can never grant the roles again.

Note: enableCollateral also doesn't set collateralActive[_collateral] = true

## Proof of Concept
https://github.com/code-423n4/2021-11-overlay/blob/914bed22f190ebe7088194453bab08c424c3f70c/contracts/mothership/OverlayV1Mothership.sol#L133-L153

```JS
 function enableCollateral (address _collateral) external onlyGovernor {
        require(collateralExists[_collateral], "OVLV1:!exists");
        require(!collateralActive[_collateral], "OVLV1:!disabled");
        OverlayToken(ovl).grantRole(OverlayToken(ovl).MINTER_ROLE(), _collateral);
        OverlayToken(ovl).grantRole(OverlayToken(ovl).BURNER_ROLE(), _collateral);
    }

    function disableCollateral (address _collateral) external onlyGovernor {
        require(collateralActive[_collateral], "OVLV1:!enabled");
        OverlayToken(ovl).revokeRole(OverlayToken(ovl).MINTER_ROLE(), _collateral);
        OverlayToken(ovl).revokeRole(OverlayToken(ovl).BURNER_ROLE(), _collateral);
    }
```

## Tools Used

## Recommended Mitigation Steps
In function enableCollateral() add the following (after the require):
collateralActive[_collateral] = true;

In function disableCollateral add the following (after the require):
collateralActive[_collateral] = false;

