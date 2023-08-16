## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed

# [Incorrect strike price displayed in name/symbol of qToken ](https://github.com/code-423n4/2022-03-rolla-findings/issues/28) 

# Lines of code

https://github.com/RollaProject/quant-protocol/blob/main/contracts/options/QTokenStringUtils.sol#L38
https://github.com/RollaProject/quant-protocol/blob/main/contracts/options/QTokenStringUtils.sol#L90
https://github.com/RollaProject/quant-protocol/blob/main/contracts/options/QTokenStringUtils.sol#L136
https://github.com/RollaProject/quant-protocol/blob/main/contracts/options/QTokenStringUtils.sol#L206


# Vulnerability details

## Impact

`_slice()` in `options/QTokenStringUtils.sol` cut a string into `string[start:end]` However, while fetching bytes, it uses `bytes(_s)[_start+1]` instead of `bytes(_s)[_start+i]`. This causes the return string to be composed of `_s[start]*(_end-_start)`. The result of this function is then used to represent the decimal part of strike price in name/symbol of qToken, leading to potential confusion over the actual value of options.


## Proof of Concept

ERC20 tokens are usually identified by their name and symbol. If the symbols are incorrect, confusions may occur. Some may argue that even if names and symbols are not accurate, it is still possible to identify correct information/usage of tokens by querying the provided view functions and looking at its interactions with other contracts. However, the truth is many users of those tokens are not very tech savvy, and it is reasonable to believe a large proportion of users are not equipped with enough knowledge, or not willing to dig further than the plain symbols and names. This highlights the importance of maintaining a correct facade for ERC20 tokens.

The bug demonstrated here shows that any qToken with decimals in its strike price will be misdisplayed, and the maximal difference between actual price and displayed one can be up to 0.1 BUSD.

The exploit can be outlined through the following steps:

* Alice created a call option with strike price 10000.90001. The expected symbol should for this qToken should be : `ROLLA WETH 31-December-2022 10000.90001 Call`

* Both `_qTokenName()` and `_qTokenSymbol()` in `options/QTokenStringUtils.sol` use `_displayedStrikePrice()` to get the strike price string which should be `10000.90001`

https://github.com/RollaProject/quant-protocol/blob/main/contracts/options/QTokenStringUtils.sol#L38
https://github.com/RollaProject/quant-protocol/blob/main/contracts/options/QTokenStringUtils.sol#L90

```
    function _qTokenName(
        address _quantConfig,
        address _underlyingAsset,
        address _strikeAsset,
        uint256 _strikePrice,
        uint256 _expiryTime,
        bool _isCall
    ) internal view virtual returns (string memory tokenName) {
        string memory underlying = _assetSymbol(_quantConfig, _underlyingAsset);
        string memory displayStrikePrice = _displayedStrikePrice(
            _strikePrice,
            _strikeAsset
        );
		
        ...
		
        tokenName = string(
            abi.encodePacked(
                "ROLLA",
                " ",
                underlying,
                " ",
                _uintToChars(day),
                "-",
                monthFull,
                "-",
                Strings.toString(year),
                " ",
                displayStrikePrice,
                " ",
                typeFull
            )
        );
    }
```

```

    function _qTokenSymbol(
        address _quantConfig,
        address _underlyingAsset,
        address _strikeAsset,
        uint256 _strikePrice,
        uint256 _expiryTime,
        bool _isCall
    ) internal view virtual returns (string memory tokenSymbol) {
        string memory underlying = _assetSymbol(_quantConfig, _underlyingAsset);
        string memory displayStrikePrice = _displayedStrikePrice(
            _strikePrice,
            _strikeAsset
        );

        // convert the expiry to a readable string
        (uint256 year, uint256 month, uint256 day) = DateTime.timestampToDate(
            _expiryTime
        );

        // get option type string
        (string memory typeSymbol, ) = _getOptionType(_isCall);

        // get option month string
        (string memory monthSymbol, ) = _getMonth(month);

        /// concatenated symbol string
        tokenSymbol = string(
            abi.encodePacked(
                "ROLLA",
                "-",
                underlying,
                "-",
                _uintToChars(day),
                monthSymbol,
                _uintToChars(year),
                "-",
                displayStrikePrice,
                "-",
                typeSymbol
            )
        );
    }
```

* `_displayedStrikePrice()` combines the quotient and the remainder to form the strike price string. The remainder use `_slice` to compute. In this case, the quotient is `10000` and the remainder is `90001`

https://github.com/RollaProject/quant-protocol/blob/main/contracts/options/QTokenStringUtils.sol#L136
```
function _displayedStrikePrice(uint256 _strikePrice, address _strikeAsset)
        internal
        view
        virtual
        returns (string memory)
    {
        uint256 strikePriceDigits = ERC20(_strikeAsset).decimals();
        uint256 strikePriceScale = 10**strikePriceDigits;
        uint256 remainder = _strikePrice % strikePriceScale;
        uint256 quotient = _strikePrice / strikePriceScale;
        string memory quotientStr = Strings.toString(quotient);

        if (remainder == 0) {
            return quotientStr;
        }

        uint256 trailingZeroes;
        while (remainder % 10 == 0) {
            remainder /= 10;
            trailingZeroes++;
        }

        // pad the number with "1 + starting zeroes"
        remainder += 10**(strikePriceDigits - trailingZeroes);

        string memory tmp = Strings.toString(remainder);
        tmp = _slice(tmp, 1, (1 + strikePriceDigits) - trailingZeroes);

        return string(abi.encodePacked(quotientStr, ".", tmp));
    }
```

* However inside the loop of `_slice()`, `slice[i] = bytes(_s)[_start + 1];` lead to an incorrect string, which is `90001`

https://github.com/RollaProject/quant-protocol/blob/main/contracts/options/QTokenStringUtils.sol#L206

```
    function _slice(
        string memory _s,
        uint256 _start,
        uint256 _end
    ) internal pure virtual returns (string memory) {
        uint256 range = _end - _start;
        bytes memory slice = new bytes(range);
        for (uint256 i = 0; i < range; ) {
            slice[i] = bytes(_s)[_start + 1];
            unchecked {
                ++i;
            }
        }

        return string(slice);
    }
```

* The final qtoken name now becomes `ROLLA WETH 31-December-2022 10000.99999 Call`, which results in confusion over the actual value of options.

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Fix the bug in the `_slice()`

```
    function _slice(
        string memory _s,
        uint256 _start,
        uint256 _end
    ) internal pure virtual returns (string memory) {
        uint256 range = _end - _start;
        bytes memory slice = new bytes(range);
        for (uint256 i = 0; i < range; ) {
            slice[i] = bytes(_s)[_start + i];
            unchecked {
                ++i;
            }
        }

        return string(slice);
    }
```


