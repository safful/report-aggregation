## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [QTokens with the same symbol will lead to mistakes](https://github.com/code-423n4/2022-03-rolla-findings/issues/38) 

# Lines of code

https://github.com/code-423n4/2022-03-rolla/blob/efe4a3c1af8d77c5dfb5ba110c3507e67a061bdd/quant-protocol/contracts/options/QTokenStringUtils.sol#L115-L130


# Vulnerability details

The `README.md` states:
> Bob can then trade the QToken with Alice for a premium. The method for doing that is beyond the scope of the protocol but can be done via any smart contract trading platform e.g. 0x.

https://github.com/code-423n4/2022-03-rolla/blob/efe4a3c1af8d77c5dfb5ba110c3507e67a061bdd/README.md?plain=1#L70

It is therefore important that tokens be easily identifiable so that trading on DEXes is not error-prone.

## Impact
Currently the `QToken` `name` includes the full year but the `QToken` symbol only contains the last two digits of the year, which can lead to mistakes. If someone mints a QToken with an expiry 100 years into the future, then the year will be truncated and appear as if the token expired this year. Normal centralized exchanges prevent this by listing options themselves and ensuring that there are never two options with the same identifier at the same time. The Rolla protocol does not have any such protections. Users must be told to not only check that the symbol name is what they expect, but to also separately check the token name or the specific expiry, or they might buy the wrong option on a DEX, or have fat-fingered during minting on a non-Rolla web interface. It's important to minimize the possibility of mistakes, and not including the full year in the symbol makes things error-prone, and will lead to other options providers winning out.

The 0x [REST interface](https://docs.0x.org/0x-api-swap/api-references/get-swap-v1-quote) for swaps has the ability to do a swap by token name rather than by token address. I was unable to figure out whether there was an allow-list of token names, or if it is easy to add a new token. If there is no, or an easily bypassed, access-control for adding new tokens, I would say this finding should be upgraded to high-severity, though I doubt this is the case.

## Proof of Concept
```solidity
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
```
https://github.com/code-423n4/2022-03-rolla/blob/efe4a3c1af8d77c5dfb5ba110c3507e67a061bdd/quant-protocol/contracts/options/QTokenStringUtils.sol#L115-L130

```solidity
    /// @return 2 characters that correspond to a number
    function _uintToChars(uint256 _number)
        internal
        pure
        virtual
        returns (string memory)
    {
        if (_number > 99) {
            _number %= 100;
        }

        string memory str = Strings.toString(_number);

        if (_number < 10) {
            return string(abi.encodePacked("0", str));
        }

        return str;
    }
```
https://github.com/code-423n4/2022-03-rolla/blob/efe4a3c1af8d77c5dfb5ba110c3507e67a061bdd/quant-protocol/contracts/options/QTokenStringUtils.sol#L181-L199

## Tools Used
Code inspection

## Recommended Mitigation Steps
Include the full year in the token's symbol


