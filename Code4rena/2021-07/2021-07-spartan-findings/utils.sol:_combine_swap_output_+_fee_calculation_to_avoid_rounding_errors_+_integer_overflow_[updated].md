## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Utils.sol: Combine Swap Output + Fee Calculation to avoid Rounding Errors + Integer Overflow [Updated]](https://github.com/code-423n4/2021-07-spartan-findings/issues/34) 

# Handle

hickuphh3


# Vulnerability details

### Impact

For minting, burning of synths and swaps, the fee and output amounts are calculated separately via `calcSwapOutput` and `calcSwapFee`. To avoid rounding errors and duplicate calculations, it would be best to combine both of these functions and return both outputs at once.

For example, if we take `x = 60000, X = 73500, Y = 50321`, the actual swap fee should be `10164.57` and output `12451.6`. However, `calcSwapOutput` and `calcSwapFee` returns `10164` and `12451`, leaving 1 wei unaccounted for. This can be avoided by combining the calculations as suggested below. The fee and actual output will be `10164` and `12452` instead.

Functions that have to call `calcSwapOutput` within the contract (eg. `calcSwapValueInBaseWithPool`) should call this function as well, for calculation consistency.

In addition, calculations for both `calcSwapOutput` and `calcSwapFee` will phantom overflow if the input values become too large. (Eg. `x = 2^128, Y=2^128`). This can be avoided by the suggested implementation below using the FullMath library.

### Recommended Mitigation Steps

```jsx
function calcSwapFeeAndOutput(uint x, uint X, uint Y) public pure returns (uint output, uint swapFee) {
		   uint xAddX = x + X;
		   uint rawOutput = FullMath.mulDiv(x, Y, xAddX);
		   swapFee = FullMath.mulDiv(rawOutput, x, xAddX);
		   output = rawOutput - swapFee;
}

function calcSwapValueInBaseWithPool(address pool, uint amount) public view returns (uint _output){
       uint _baseAmount = iPOOL(pool).baseAmount();
       uint _tokenAmount = iPOOL(pool).tokenAmount();
       (_output, ) = calcSwapFeeAndOutput(amount, _tokenAmount, _baseAmount);
}
```

The FullMath library is included (and made compatible with sol 0.8+) below for convenience.

```jsx
// SPDX-License-Identifier: MIT
pragma solidity >= 0.8.0;

/// @title Contains 512-bit math functions
/// @notice Facilitates multiplication and division that can have overflow of an intermediate value without any loss of precision
/// @dev Handles "phantom overflow" i.e., allows multiplication and division where an intermediate value overflows 256 bits
library FullMath {
    /// @notice Calculates floor(a×b÷denominator) with full precision. Throws if result overflows a uint256 or denominator == 0
    /// @param a The multiplicand
    /// @param b The multiplier
    /// @param denominator The divisor
    /// @return result The 256-bit result
    /// @dev Credit to Remco Bloemen under MIT license https://xn--2-umb.com/21/muldiv
    function mulDiv(
        uint256 a,
        uint256 b,
        uint256 denominator
    ) internal pure returns (uint256 result) {
        // 512-bit multiply [prod1 prod0] = a * b
        // Compute the product mod 2**256 and mod 2**256 - 1
        // then use the Chinese Remainder Theorem to reconstruct
        // the 512 bit result. The result is stored in two 256
        // variables such that product = prod1 * 2**256 + prod0
        uint256 prod0; // Least significant 256 bits of the product
        uint256 prod1; // Most significant 256 bits of the product
        assembly {
            let mm := mulmod(a, b, not(0))
            prod0 := mul(a, b)
            prod1 := sub(sub(mm, prod0), lt(mm, prod0))
        }

        // Handle non-overflow cases, 256 by 256 division
        if (prod1 == 0) {
            require(denominator > 0, "0 denom");
            assembly {
                result := div(prod0, denominator)
            }
            return result;
        }

        // Make sure the result is less than 2**256.
        // Also prevents denominator == 0
        require(denominator > prod1, "denom <= prod1");

        ///////////////////////////////////////////////
        // 512 by 256 division.
        ///////////////////////////////////////////////

        // Make division exact by subtracting the remainder from [prod1 prod0]
        // Compute remainder using mulmod
        uint256 remainder;
        assembly {
            remainder := mulmod(a, b, denominator)
        }
        // Subtract 256 bit number from 512 bit number
        assembly {
            prod1 := sub(prod1, gt(remainder, prod0))
            prod0 := sub(prod0, remainder)
        }

        // Factor powers of two out of denominator
        // Compute largest power of two divisor of denominator.
        // Always >= 1.
        uint256 twos = denominator & (~denominator + 1);
        // Divide denominator by power of two
        assembly {
            denominator := div(denominator, twos)
        }

        // Divide [prod1 prod0] by the factors of two
        assembly {
            prod0 := div(prod0, twos)
        }
        // Shift in bits from prod1 into prod0. For this we need
        // to flip `twos` such that it is 2**256 / twos.
        // If twos is zero, then it becomes one
        assembly {
            twos := add(div(sub(0, twos), twos), 1)
        }
        unchecked {
            prod0 |= prod1 * twos;
            
            // Invert denominator mod 2**256
            // Now that denominator is an odd number, it has an inverse
            // modulo 2**256 such that denominator * inv = 1 mod 2**256.
            // Compute the inverse by starting with a seed that is correct
            // correct for four bits. That is, denominator * inv = 1 mod 2**4
            uint256 inv = (3 * denominator) ^ 2;
            
            // Now use Newton-Raphson iteration to improve the precision.
            // Thanks to Hensel's lifting lemma, this also works in modular
            // arithmetic, doubling the correct bits in each step.
            inv *= 2 - denominator * inv; // inverse mod 2**8
            inv *= 2 - denominator * inv; // inverse mod 2**16
            inv *= 2 - denominator * inv; // inverse mod 2**32
            inv *= 2 - denominator * inv; // inverse mod 2**64
            inv *= 2 - denominator * inv; // inverse mod 2**128
            inv *= 2 - denominator * inv; // inverse mod 2**256
            
            // Because the division is now exact we can divide by multiplying
            // with the modular inverse of denominator. This will give us the
            // correct result modulo 2**256. Since the precoditions guarantee
            // that the outcome is less than 2**256, this is the final result.
            // We don't need to compute the high bits of the result and prod1
            // is no longer required.
            result = prod0 * inv;
        }
        return result;
    }
}
```

