## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- M-04

# [`CidNFT`: Broken `tokenURI` function](https://github.com/code-423n4/2023-01-canto-identity-findings/issues/89) 

# Lines of code

https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L140


# Vulnerability details

[`CidNFT#tokenURI`](https://github.com/code-423n4/2023-01-canto-identity/blob/dff8e74c54471f5f3b84c217848234d474477d82/src/CidNFT.sol#L133-L142) does not convert the `uint256 _id` argument to a string before interpolating it in the token URI:

```solidity
    /// @notice Get the token URI for the provided ID
    /// @param _id ID to retrieve the URI for
    /// @return tokenURI The URI of the queried token (path to a JSON file)
    function tokenURI(uint256 _id) public view override returns (string memory) {
        if (ownerOf[_id] == address(0))
            // According to ERC721, this revert for non-existing tokens is required
            revert TokenNotMinted(_id);
        return string(abi.encodePacked(baseURI, _id, ".json"));
    }

```

This means the raw bytes of the 32-byte ABI encoded integer `_id` will be interpolated into the token URI, e.g. `0x0000000000000000000000000000000000000000000000000000000000000001` for ID #1. 

Most of the resulting UTF-8 strings will be malformed, incorrect, or invalid URIs. For example, token ID #1 will show up as the invisible "start of heading" control character, and ID #42 will show as the asterisk symbol `*`. URI-unsafe characters will break the token URIs altogether.

**Impact**
- `CidNFT` tokens will have invalid `tokenURI`s. Offchain tools that read the `tokenURI` view may break or display malformed data.

**Suggestion**

Convert the `_id` to a string before calling `abi.encodePacked`. Latest Solmate includes a `LibString` helper library for this purpose:

```solidity
    import "solmate/utils/LibString.sol";

    /// @notice Get the token URI for the provided ID
    /// @param _id ID to retrieve the URI for
    /// @return tokenURI The URI of the queried token (path to a JSON file)
    function tokenURI(uint256 _id) public view override returns (string memory) {
        if (ownerOf[_id] == address(0))
            // According to ERC721, this revert for non-existing tokens is required
            revert TokenNotMinted(_id);
        return string(abi.encodePacked(baseURI, LibString.toString(_id), ".json"));
    }

```

**Test case**

```solidity
    function test_InvalidTokenURI() public {
        uint256 id1 = cidNFT.numMinted() + 1;
        uint256 id2 = cidNFT.numMinted() + 2;
        // mint id1
        cidNFT.mint(new bytes[](0));
        // mint id2
        cidNFT.mint(new bytes[](0));

        // These pass — the raw bytes '0000000000000000000000000000000000000000000000000000000000000001' are interpolated as _id.
        assertEq(string(bytes(hex"7462643a2f2f626173655f7572692f00000000000000000000000000000000000000000000000000000000000000012e6a736f6e")), cidNFT.tokenURI(id1));
        assertEq(string(bytes(hex"7462643a2f2f626173655f7572692f00000000000000000000000000000000000000000000000000000000000000022e6a736f6e")), cidNFT.tokenURI(id2));

        // These fail - the generated string on the right is not the expected string on the left. 
        assertEq("tbd://base_uri/1.json", cidNFT.tokenURI(id1));
        assertEq("tbd://base_uri/2.json", cidNFT.tokenURI(id2));
    }
```