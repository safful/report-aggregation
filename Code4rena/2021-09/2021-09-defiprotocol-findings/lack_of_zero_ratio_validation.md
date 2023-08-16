## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Lack of zero ratio validation](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/83) 

# Handle

defsec


# Vulnerability details

## Impact

During the manual code review, It has been observed that zero value has not been checked on that "ibRatio" variable. That can cause  miscalculation of the liquidity.


## Proof of Concept

1. Navigate to "https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Basket.sol"
2. Go to the line #217.

"""
        ibRatio = newRatio;
"""

3. Onlyauction modifier can assign ibRation to 0.
4. That can affect tokenAmount on the function. 

"""
    function pushUnderlying(uint256 amount, address to) private {
        for (uint256 i = 0; i < weights.length; i++) {
            uint256 tokenAmount = amount * weights[i] * ibRatio / BASE / BASE;
            IERC20(tokens[i]).safeTransfer(to, tokenAmount);
        }
    }
"""

## Tools Used

None

## Recommended Mitigation Steps

Validate to ibRatio variable is more than zero.

"""
require(ibRation > 0 , "ibRatio should be more than zero");
"""

