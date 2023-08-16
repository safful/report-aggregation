## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-02-nested-findings/issues/72) 

Gas optimization

1 Use default value for uint256 and use ++i instead of i++ in for loop

https://github.com/code-423n4/2022-02-nested/blob/main/contracts/FeeSplitter.sol#L136-L139
https://github.com/code-423n4/2022-02-nested/blob/main/contracts/FeeSplitter.sol#L148
https://github.com/code-423n4/2022-02-nested/blob/main/contracts/FeeSplitter.sol#L165
https://github.com/code-423n4/2022-02-nested/blob/main/contracts/FeeSplitter.sol#L261
https://github.com/code-423n4/2022-02-nested/blob/main/contracts/FeeSplitter.sol#L280
https://github.com/code-423n4/2022-02-nested/blob/main/contracts/FeeSplitter.sol#L318

https://github.com/code-423n4/2022-02-nested/blob/main/contracts/NestedFactory.sol#L103
https://github.com/code-423n4/2022-02-nested/blob/main/contracts/NestedFactory.sol#L113
https://github.com/code-423n4/2022-02-nested/blob/main/contracts/NestedFactory.sol#L153
https://github.com/code-423n4/2022-02-nested/blob/main/contracts/NestedFactory.sol#L213
https://github.com/code-423n4/2022-02-nested/blob/main/contracts/NestedFactory.sol#L273
https://github.com/code-423n4/2022-02-nested/blob/main/contracts/NestedFactory.sol#L327
https://github.com/code-423n4/2022-02-nested/blob/main/contracts/NestedFactory.sol#L369
https://github.com/code-423n4/2022-02-nested/blob/main/contracts/NestedFactory.sol#L581

https://github.com/code-423n4/2022-02-nested/blob/main/contracts/NestedFactory.sol#L581

https://github.com/code-423n4/2022-02-nested/blob/main/contracts/OperatorResolver.sol#L40
https://github.com/code-423n4/2022-02-nested/blob/main/contracts/OperatorResolver.sol#L75


Feesplitter.sol

2 Use storage for shareholders[_accountIndex]saves gas in updateShareholder.

https://github.com/code-423n4/2022-02-nested/blob/main/contracts/FeeSplitter.sol#L134-L140

function updateShareholder(uint256 _accountIndex, uint96 _weight) external onlyOwner {
        require(_accountIndex < shareholders.length, "FS: INVALID_ACCOUNT_INDEX");
        Shareholder storage _shareholder = shareholders[_accountIndex];
        totalWeights = totalWeights + _weight - _shareholder.weight;
        require(totalWeights != 0, "FS: TOTAL_WEIGHTS_ZERO");
        _shareholder.weight = _weight;
        emit ShareholderUpdated(_shareholder.account, _weight);
    }

NestedRecords.sol

3 Check _reserve != address(0) earlier in store save gas.

https://github.com/code-423n4/2022-02-nested/blob/main/contracts/NestedRecords.sol#L111-L127

require(_reserve != address(0), “NRC: NO_ADDRESS”); must be checked separately at the beginning of function.

4 Use storage for records[_nftId] saves gas in store.

https://github.com/code-423n4/2022-02-nested/blob/main/contracts/NestedRecords.sol#L111-L132

function store(
        uint256 _nftId,
        address _token,
        uint256 _amount,
        address _reserve
    ) external onlyFactory {
        NftRecord storage _record = records[_nftId];
        uint256 amount = _record.holdings[_token];
        // uint256 amount = records[_nftId].holdings[_token];
        if (amount != 0) {
            require(_record.reserve == _reserve, "NRC: RESERVE_MISMATCH");
            updateHoldingAmount(_nftId, _token, amount + _amount);
            return;
        }
        require(_record.tokens.length < maxHoldingsCount, "NRC: TOO_MANY_TOKENS");
        require(
            _reserve != address(0) && (_reserve == _record.reserve || _record.reserve == address(0)),
            "NRC: INVALID_RESERVE"
        );
 
        _record.holdings[_token] = _amount;
        _record.tokens.push(_token);
        _record.reserve = _reserve;
    }
 
