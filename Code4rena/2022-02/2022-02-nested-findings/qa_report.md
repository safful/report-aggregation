## Tags

- bug
- QA (Quality Assurance)
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-02-nested-findings/issues/70) 

# QA Report  
**Table of Contents:**  
- [QA Report](#qa-report)
  - [Foreword](#foreword)
  - [Comparisons](#comparisons)
    - [Comparison that should be inclusive in NestedRecords.sol](#comparison-that-should-be-inclusive-in-nestedrecordssol)
  - [Variables](#variables)
    - [Missing Address(0) checks](#missing-address0-checks)
    - [Check if a value is in an array before a push](#check-if-a-value-is-in-an-array-before-a-push)
    - [Variables that should be grouped together in a struct](#variables-that-should-be-grouped-together-in-a-struct)
      - [File: FeeSplitter.sol](#file-feesplittersol)
  - [Arithmetics](#arithmetics)
    - [Possible division by 0](#possible-division-by-0)
  - [Revert Strings](#revert-strings)
    - [File: NestedFactory.sol](#file-nestedfactorysol)
      - [Inconsistent Revert string](#inconsistent-revert-string)
    - [File: MixinOperatorResolver.sol](#file-mixinoperatorresolversol)
      - [Inconsistent Revert string (1)](#inconsistent-revert-string-1)
      - [Misleading + Inconsistent Revert string (2)](#misleading--inconsistent-revert-string-2)
    - [File: OwnableProxyDelegation.sol](#file-ownableproxydelegationsol)
      - [Inconsistent Revert string (1)](#inconsistent-revert-string-1-1)
      - [Inconsistent Revert string (2)](#inconsistent-revert-string-2)
    - [File: ZeroExOperator.sol](#file-zeroexoperatorsol)
      - [Inconsistent Revert string (1)](#inconsistent-revert-string-1-2)
      - [Inconsistent Revert string (2)](#inconsistent-revert-string-2-1)
  - [Comments](#comments)
    - [File: NestedFactory.sol](#file-nestedfactorysol-1)
      - [Missing comment "@return" (1)](#missing-comment-return-1)
      - [Missing comment "@return" (2)](#missing-comment-return-2)
      - [Missing comment "@param" (1)](#missing-comment-param-1)
    - [File: NestedRecords.sol](#file-nestedrecordssol)
      - [Missing comment "@return" (1)](#missing-comment-return-1-1)
      - [Missing comment "@return" (2)](#missing-comment-return-2-1)
      - [Misleading comment on "@return"](#misleading-comment-on-return)
    - [File: MixinOperatorResolver.sol](#file-mixinoperatorresolversol-1)
      - [Missing 2 comments "@param" & changeable "@return" comment/variable](#missing-2-comments-param--changeable-return-commentvariable)
    - [File: ExchangeHelpers.sol](#file-exchangehelperssol)
      - [Missing comment "@return"](#missing-comment-return)

## Foreword
- **`@audit` tags**
> The code is annotated at multiple places with `//@audit` comments to pinpoint the issues. Please, pay attention to them for more details.

## Comparisons
### Comparison that should be inclusive in NestedRecords.sol
```
File: NestedRecords.sol
123:         require(records[_nftId].tokens.length < maxHoldingsCount, "NRC: TOO_MANY_TOKENS"); //@audit should be inclusive
```

As length isn't 0 indexed, I believe, as an example to illustrate, that if `maxHoldingsCount == 1`, then `records[_nftId].tokens.length == 1` should be a passing condition. Therefore, I suggest changing `<` with `<=`

## Variables  
### Missing Address(0) checks  
```
File: MixinOperatorResolver.sol
22:     constructor(address _resolver) {
23:         resolver = OperatorResolver(_resolver); //@audit missing address(0) check on immutable just like in the constructors in FeeSplitter.sol and NestedFactory.sol
24:     }
```

### Check if a value is in an array before a push
In `NestedRecords.sol`'s `store` function, it's possible to push an existing `address _token` several times in the same array 
```
File: NestedRecords.sol
130:         records[_nftId].tokens.push(_token); //@audit : should check existence
```
The previous lines of codes don't prevent this.
The `store` function has the modifier `onlyFactory` and the only impact seem to be a possible maximization of `records[_nftId].tokens.length` (so that it reaches `maxHoldingsCount`).

### Variables that should be grouped together in a struct  
For maps that use the same key value: having separate fields is error prone (like in case of deletion or future new fields).  
By regrouping, it's then possible to delete all related fields with a simple `delete newStruct[previousSameKeyForAllPreviousMaps]`.
  
#### File: FeeSplitter.sol
2 maps can be grouped together, as they use the same `_account` key:  
```  
62:     struct TokenRecords {
63:         uint256 totalShares;
64:         uint256 totalReleased;
65:         mapping(address => uint256) shares; //@audit group 
66:         mapping(address => uint256) released; //@audit group 
67:     }
```  
I'd suggest these 2 related data get grouped in a struct, let's name it `AccountInfo`:   
```  
struct AccountInfo {   
    uint256 shares;   
    uint256 released;   
}   
```  
And it would be used in this manner (where `address` is `_account`):   
```   
    struct TokenRecords {
        uint256 totalShares;
        uint256 totalReleased;
        mapping(address => AccountInfo) accountInfo;
    }   
```   
## Arithmetics  
### Possible division by 0  
There are no checks that the denominator is `!= 0` here:  
```  
File: FeeSplitter.sol
327:     function _computeShareCount(
328:         uint256 _amount,
329:         uint256 _weight,
330:         uint256 _totalWeights
331:     ) private pure returns (uint256) {
332:         return (_amount * _weight) / _totalWeights; // @audit _totalWeights can be equal to 0, see FeeSplitter.sol:L184
333:     }
```

## Revert Strings
### File: NestedFactory.sol
#### Inconsistent Revert string
```
44:         require(_exists(_tokenId), "URI query for nonexistent token");
```
All other revert strings in `NestedAsset.sol` begin with `NA: `. Only this one doesn't. It's possible to gain consistency and still have an < 32 bytes size string with the following: `"NA: URI query - inexistent token"`

### File: MixinOperatorResolver.sol
#### Inconsistent Revert string (1)
```
100:             require(tokens[0] == _outputToken, "OH: INVALID_OUTPUT_TOKEN");//@audit LOW comment : MOR like above
```
Here, `"OH: INVALID_OUTPUT_TOKEN"` should be replaced with `"MOR: INVALID_OUTPUT_TOKEN"`

#### Misleading + Inconsistent Revert string (2)
```
101:             require(tokens[1] == _inputToken, "OH: INVALID_OUTPUT_TOKEN"); //@audit LOW comment : INVALID_INPUT_TOKEN //@audit LOW comment : MOR
```
Here, `"OH: INVALID_OUTPUT_TOKEN"` should be replaced with `"MOR: INVALID_INPUT_TOKEN"`

### File: OwnableProxyDelegation.sol
#### Inconsistent Revert string (1)
```
25:         require(!initialized, "OFP: INITIALIZED"); //@audit low OFP doesn't make sense, use OPD instead (example: OwnableFactoryHandler is OFH, MixinOperatorResolver is MOR)
```
Is most contracts, the capital letters from the contract's name are used as a prefix in the revert strings (`OwnableFactoryHandler` has `OFH`, `MixinOperatorResolver` has `MOR`). Here, `OFP` doesn't really reflect `OwnableProxyDelegation`. It should be `OPD`.

#### Inconsistent Revert string (2)
```
26:         require(StorageSlot.getAddressSlot(_ADMIN_SLOT).value == msg.sender, "OFP: FORBIDDEN");//@audit should be "OPD: FORBIDDEN" 
```
Same as above: `OFP` should be `OPD`.

### File: ZeroExOperator.sol
#### Inconsistent Revert string (1)
```
32:         require(success, "ZEO: SWAP_FAILED");
...
36:         require(amountBought != 0, "ZeroExOperator::performSwap: amountBought cant be zero"); //@audit LOW do like line 32 : "ZEO: amountBought cant be zero" < 32 bytes & consistent
```
As said before, the capital letters from the contract's name are used as a prefix in the revert strings. Here, the revert string's size is > 32 bytes and isn't using the same style as 4 lines above it. `ZeroExOperator::performSwap` should be `ZEO`.

#### Inconsistent Revert string (2)
```
32:         require(success, "ZEO: SWAP_FAILED");
...
37:         require(amountSold != 0, "ZeroExOperator::performSwap: amountSold cant be zero");//@audit do like line 32 : "ZEO: amountSold cant be zero" < 32 bytes & consistent
```
Same as above: `ZeroExOperator::performSwap` should be `ZEO`.

## Comments
### File: NestedFactory.sol
#### Missing comment "@return" (1)
```
403:     /// @dev Call the operator to submit the order and add the output
404:     /// assets to the reserve (if needed).
405:     /// @param _inputToken Token used to make the orders
406:     /// @param _outputToken Expected output token
407:     /// @param _nftId The nftId
408:     /// @param _order The order calldata
409:     /// @param _toReserve True if the output is store in the reserve/records, false if not. //@audit missing @return 
410:     function _submitOrder(
411:         address _inputToken,
412:         address _outputToken,
413:         uint256 _nftId,
414:         Order calldata _order,
415:         bool _toReserve
416:     ) private returns (uint256 amountSpent) {
```

#### Missing comment "@return" (2)
```
474:     /// @dev Choose between ERC20 (safeTransfer) and ETH (deposit), to transfer from the Reserve
475:     ///      or the user wallet, to the factory.
476:     /// @param _nftId The NFT id
477:     /// @param _inputToken The token to receive
478:     /// @param _inputTokenAmount Amount to transfer
479:     /// @param _fromReserve True to transfer from the reserve
480:     /// @return Token transfered (in case of ETH)
481:     ///         The real amount received after the transfer to the factory //@audit missing @return (not the description, just the keyword)
482:     function _transferInputTokens(
483:         uint256 _nftId,
484:         IERC20 _inputToken,
485:         uint256 _inputTokenAmount,
486:         bool _fromReserve
487:     ) private returns (IERC20, uint256) {
```

#### Missing comment "@param" (1)
```
562:     /// @dev Transfer from factory and collect fees
563:     /// @param _token The token to transfer
564:     /// @param _amount The amount (with fees) to transfer
565:     /// @param _dest The address receiving the funds //@audit missing @param
566:     function _safeTransferWithFees(
567:         IERC20 _token,
568:         uint256 _amount,
569:         address _dest,
570:         uint256 _nftId
571:     ) private {
```

### File: NestedRecords.sol
#### Missing comment "@return" (1)
```
162:     /// @param _nftId The id of the NFT> //@audit missing @return
163:     function getAssetTokens(uint256 _nftId) public view returns (address[] memory) {
```

#### Missing comment "@return" (2)
```
183:     /// @param _token The address of the token //@audit missing @return
184:     function getAssetHolding(uint256 _nftId, address _token) public view returns (uint256) {
```

#### Misleading comment on "@return"
Here, the comment `@return The holdings`, which is the unique `@return` comment, suggests a returned `mapping(address => uint256) holdings` as seen on `struct NftRecord`. However, the function is actually returning a `uint256[]` and an `address[]`. Therefore, two `@return` are required and the previous one should be deleted.

Code:
```
188:     /// @notice Returns the holdings associated to a NestedAsset
189:     /// @param _nftId the id of the NestedAsset
190:     /// @return The holdings //@audit "The holdings" suggests a "mapping(address => uint256)" but a uint256[] and an address[] are returned.  
191:     function tokenHoldings(uint256 _nftId) public view returns (address[] memory, uint256[] memory) {
```

### File: MixinOperatorResolver.sol
#### Missing 2 comments "@param" & changeable "@return" comment/variable
```
    /// @dev Build the calldata (with safe datas) and call the Operator
    /// @param _order The order to execute //@audit missing @param _inputToken and @param _outputToken
    /// @return success If the operator call is successful
    /// @return amounts The amounts from the execution (used and received) //@audit why not use uint256[2]?
    ///         - amounts[0] : The amount of output token
    ///         - amounts[1] : The amount of input token USED by the operator (can be different than expected)
    function callOperator(
        INestedFactory.Order calldata _order,
        address _inputToken,
        address _outputToken
    ) internal returns (bool success, uint256[] memory amounts) {
```
I suggest changing the returned uint256[] to uint256[2]

### File: ExchangeHelpers.sol
#### Missing comment "@return"
```
10:     /// @dev Perform a swap between two tokens
11:     /// @param _sellToken Token to exchange
12:     /// @param _swapTarget The address of the contract that swaps tokens
13:     /// @param _swapCallData Call data provided by 0x to fill the quote //@audit missing @return
14:     function fillQuote(
15:         IERC20 _sellToken,
16:         address _swapTarget,
17:         bytes memory _swapCallData
18:     ) internal returns (bool) {
```