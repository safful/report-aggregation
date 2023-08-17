## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-06-nibbl-findings/issues/282) 

## GAS

1.
## Title: Caching _tokens[i] can save gas

Occurrences:
https://github.com/code-423n4/2022-06-nibbl/blob/main/contracts/Basket.sol#L71-L73
https://github.com/code-423n4/2022-06-nibbl/blob/main/contracts/Basket.sol#L44-L45


Instead of calling array value by assign its key every call, we can cache it for gas saving
Change to:
```
        for (uint256 i = 0; i < _tokens.length; i++) {
	    address _token = _tokens[i];
	    uint _tokenId = _tokenIds[i]
            uint256 _balance = IERC1155(_token).balanceOf(address(this),  _tokenId);
            IERC1155(_token).safeTransferFrom(address(this), _to, _tokenId, _balance, "0");
            emit WithdrawERC1155(_token, _tokenId, _balance, _to);
        }
```
By doing this way we can save 48 gas per call


2.
## Title: Prefix increment and unchecked for `i`

Occurrences:
https://github.com/code-423n4/2022-06-nibbl/blob/main/contracts/Basket.sol#L43
https://github.com/code-423n4/2022-06-nibbl/blob/main/contracts/Basket.sol#L70
https://github.com/code-423n4/2022-06-nibbl/blob/main/contracts/Basket.sol#L93

Best practice for doing increment is using prefix increment and unchecked for `i` var inside for():

Change to:
```
        for (uint256 i = 0; i < _tokens.length;) {
            IERC721(_tokens[i]).safeTransferFrom(address(this), _to, _tokenId[i]);
            emit WithdrawERC721(_tokens[i], _tokenId[i], _to);
	    unchecked{++i;}
        }
```


3.
## Title: Using calldata to store argument variable:

Occurrences:
https://github.com/code-423n4/2022-06-nibbl/blob/main/contracts/Basket.sol#L41
https://github.com/code-423n4/2022-06-nibbl/blob/main/contracts/Basket.sol#L68

We can store `_tokens` and `_tokenIds` read only parameter with calldata instead of using memory.

## Recommended mitigation step
Change to:
```
    function withdrawMultipleERC721(address[] calldata _tokens, uint256[] calldata _tokenId, address _to) external override {
```


4.
## Title: Using < operator instead of <=

https://github.com/code-423n4/2022-06-nibbl/blob/main/contracts/NibblVault.sol#L147

By using < operator to validate we can save 3 gas per call. The 1 second difference can be ignore


5.
## Title: Using delete statement to set to default value

https://github.com/code-423n4/2022-06-nibbl/blob/main/contracts/NibblVault.sol#L477

Using delete statement to set value to 0 can save 8 gas per execution

## Recommended mitigation step
```
delete feeAccruedCurator;
```

