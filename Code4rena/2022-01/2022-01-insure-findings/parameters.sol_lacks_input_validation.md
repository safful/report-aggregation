## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Parameters.sol lacks input validation](https://github.com/code-423n4/2022-01-insure-findings/issues/44) 

# Handle

cccz


# Vulnerability details

## Impact
When setting parameters in the Parameters contract, the input parameters are not verified.

For example, in the setFeeRate function, the _target parameter is not limited. When _target is greater than 1e6, DOS will occur when used in the insure function of the PoolTemplate contract
```
    function setFeeRate(address _address, uint256 _target)
        external
        override
        onlyOwner
    {
        _fee[_address] = _target;
        emit FeeRateSet(_address, _target);
    }
   ...
   function insure(
        uint256 _amount,
        uint256 _maxCost,
        uint256 _span,
        bytes32 _target
    ) external returns (uint256) {
        //Distribute premium and fee
        uint256 _endTime = _span + block.timestamp;
        uint256 _premium = getPremium(_amount, _span);
        uint256 _fee = parameters.getFeeRate(msg.sender);

        require(
            _amount <= availableBalance(),
            "ERROR: INSURE_EXCEEDED_AVAILABLE_BALANCE"
        );
        require(_premium <= _maxCost, "ERROR: INSURE_EXCEEDED_MAX_COST");
        require(_span <= 365 days, "ERROR: INSURE_EXCEEDED_MAX_SPAN");
        require(
            parameters.getMinDate(msg.sender) <= _span,
            "ERROR: INSURE_SPAN_BELOW_MIN"
        );

        require(
            marketStatus == MarketStatus.Trading,
            "ERROR: INSURE_MARKET_PENDING"
        );
        require(paused == false, "ERROR: INSURE_MARKET_PAUSED");

        //current liquidity
        uint256 _liquidity = totalLiquidity();
        uint256 _totalCredit = totalCredit;

        //accrue premium/fee
        uint256[2] memory _newAttribution = vault.addValueBatch(
            _premium,
            msg.sender,
            [address(this), parameters.getOwner()],
            [MAGIC_SCALE_1E6-_fee, _fee]
        );
```
## Proof of Concept

https://github.com/code-423n4/2022-01-insure/blob/main/contracts/Parameters.sol

## Tools Used

Manual analysis


## Recommended Mitigation Steps

When setting parameters in the Parameters contract, verify the input parameters

