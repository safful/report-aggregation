## Tags

- bug
- sponsor confirmed
- 1 (Low Risk)
- resolved

# [Scaling factors for token 0/1 might swap in TempusAMM constructor.](https://github.com/code-423n4/2021-10-tempus-findings/issues/7) 

# Handle

chenyu


# Vulnerability details

## Impact
In TempusAMM constructor [L138](https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/amm/TempusAMM.sol#L138), the scaling factor 0 always maps to yieldShare, and scaling factor 1 always maps to principalShare, even though in L134 the two token might swap if principalShare < yieldShare, which makes _token0 = principalShare and _token1 = yieldShare, but scaling factor 0 is based on yieldShare.

Later [_scalingFactor](https://github.com/code-423n4/2021-10-tempus/blob/63f7639aad08f2bba717830ed81e0649f7fc23ee/contracts/amm/TempusAMM.sol#L802) is based on _token0, so it might get the wrong scaling factor if principalShare and yieldShare had swapped.

## Recommended Mitigation Steps
Update the lines to
```
        _scalingFactor0 = _computeScalingFactor(IERC20(address(_token0)));
        _scalingFactor1 = _computeScalingFactor(IERC20(address(_token1)));
```

