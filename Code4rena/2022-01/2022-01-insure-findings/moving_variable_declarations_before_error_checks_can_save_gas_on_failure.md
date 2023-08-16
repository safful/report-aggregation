## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Moving Variable Declarations Before Error Checks Can Save Gas on Failure](https://github.com/code-423n4/2022-01-insure-findings/issues/170) 

# Handle

0xngndev


# Vulnerability details

## Impact

In `PoolTemplate.sol` there are multiple instances where variables are declared before the error checks of the functions. In cases where a function reverts due to these error checks, that extra computation of calculating the variable being declared can be avoided by simply moving the declaration after the error checks.

Here are all the functions I found where this can be applied:

- `withdraw` function: [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L293](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L293)
- `withdrawCredit` function: [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L416](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L416)
- `insure` function: [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L465](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L465)
- `reedem` function: [https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L548](https://github.com/code-423n4/2022-01-insure/blob/main/contracts/PoolTemplate.sol#L548)

## Recommended Mitigation Steps

- Change `withdraw` function to:

```solidity
function withdraw(uint256 _amount) external returns (uint256 _retVal) {
  require(
      marketStatus == MarketStatus.Trading,
      "ERROR: WITHDRAWAL_PENDING"
  );
  require(
      withdrawalReq[msg.sender].timestamp +
          parameters.getLockup(msg.sender) <
          block.timestamp,
      "ERROR: WITHDRAWAL_QUEUE"
  );
  require(
      withdrawalReq[msg.sender].timestamp +
          parameters.getLockup(msg.sender) +
          parameters.getWithdrawable(msg.sender) >
          block.timestamp,
      "ERROR: WITHDRAWAL_NO_ACTIVE_REQUEST"
  );
  require(
      withdrawalReq[msg.sender].amount >= _amount,
      "ERROR: WITHDRAWAL_EXCEEDED_REQUEST"
  );
  require(_amount > 0, "ERROR: WITHDRAWAL_ZERO");
  require(
      _retVal <= availableBalance(),
      "ERROR: WITHDRAW_INSUFFICIENT_LIQUIDITY"
  );

  uint256 _supply = totalSupply();
  require(_supply != 0, "ERROR: NO_AVAILABLE_LIQUIDITY");

  uint256 _liquidity = originalLiquidity();
  _retVal = (_amount * _liquidity) / _supply;

  //reduce requested amount
  withdrawalReq[msg.sender].amount -= _amount;

  //Burn iToken
  _burn(msg.sender, _amount);

  //Withdraw liquidity
  vault.withdrawValue(_retVal, msg.sender);

  emit Withdraw(msg.sender, _amount, _retVal);
}
```

- Change `withdrawCredit` function to:

```solidity
function withdrawCredit(uint256 _credit)
        external
        override
        returns (uint256 _pending)
    {
      IndexInfo storage _index = indicies[msg.sender];
      require(
          IRegistry(registry).isListed(msg.sender) &&
              _index.credit >= _credit &&
              _credit <= availableBalance(),
          "ERROR: WITHDRAW_CREDIT_BAD_CONDITIONS"
      );

      uint256 _rewardPerCredit = rewardPerCredit;

      //calculate acrrued premium
      _pending = _sub(
          (_index.credit * _rewardPerCredit) / MAGIC_SCALE_1E6,
          _index.rewardDebt
      );

      //Withdraw liquidity
      if (_credit > 0) {
          totalCredit -= _credit;
          _index.credit -= _credit;
          emit CreditDecrease(msg.sender, _credit);
      }

      //withdraw acrrued premium
      if (_pending > 0) {
          vault.transferAttribution(_pending, msg.sender);
          attributionDebt -= _pending;
          _index.rewardDebt =
              (_index.credit * _rewardPerCredit) /
              MAGIC_SCALE_1E6;
      }
}
```

- Change `insure` function to:

```solidity
function insure(
        uint256 _amount,
        uint256 _maxCost,
        uint256 _span,
        bytes32 _target
    ) external returns (uint256) {
      //Distribute premium and fee
      uint256 _premium = getPremium(_amount, _span);

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

      uint256 _endTime = _span + block.timestamp;
      uint256 _fee = parameters.getFeeRate(msg.sender);

      //current liquidity
      uint256 _liquidity = totalLiquidity();
      uint256 _totalCredit = totalCredit;

      //accrue premium/fee
      uint256[2] memory _newAttribution = vault.addValueBatch(
          _premium,
          msg.sender,
          [address(this), parameters.getOwner()],
          [MAGIC_SCALE_1E6 - _fee, _fee]
      );

      //Lock covered amount
      uint256 _id = allInsuranceCount;
      lockedAmount += _amount;
      Insurance memory _insurance = Insurance(
          _id,
          block.timestamp,
          _endTime,
          _amount,
          _target,
          msg.sender,
          true
      );
      insurances[_id] = _insurance;
      allInsuranceCount += 1;

      //Calculate liquidity for index
      if (_totalCredit > 0) {
          uint256 _attributionForIndex = (_newAttribution[0] * _totalCredit) /
              _liquidity;
          attributionDebt += _attributionForIndex;
          rewardPerCredit += ((_attributionForIndex * MAGIC_SCALE_1E6) /
              _totalCredit);
      }

      emit Insured(
          _id,
          _amount,
          _target,
          block.timestamp,
          _endTime,
          msg.sender,
          _premium
      );

        return _id;
  }
```

- Change `redeem` function to:

```solidity
function redeem(uint256 _id, bytes32[] calldata _merkleProof) external {
      require(
          marketStatus == MarketStatus.Payingout,
          "ERROR: NO_APPLICABLE_INCIDENT"
      );
      Insurance storage _insurance = insurances[_id];
      require(_insurance.status == true, "ERROR: INSURANCE_NOT_ACTIVE");
      require(_insurance.insured == msg.sender, "ERROR: NOT_YOUR_INSURANCE");
      uint256 _incidentTimestamp = incident.incidentTimestamp;
      require(
          marketStatus == MarketStatus.Payingout &&
              _insurance.startTime <= _incidentTimestamp &&
              _insurance.endTime >= _incidentTimestamp,
          "ERROR: INSURANCE_NOT_APPLICABLE"
      );
      bytes32 _targets = incident.merkleRoot;
      require(
          MerkleProof.verify(
              _merkleProof,
              _targets,
              keccak256(
                  abi.encodePacked(_insurance.target, _insurance.insured)
              )
          ) ||
              MerkleProof.verify(
                  _merkleProof,
                  _targets,
                  keccak256(abi.encodePacked(_insurance.target, address(0)))
              ),
          "ERROR: INSURANCE_EXEMPTED"
      );
      uint256 _payoutNumerator = incident.payoutNumerator;
      uint256 _payoutDenominator = incident.payoutDenominator;

      _insurance.status = false;
      lockedAmount -= _insurance.amount;

      uint256 _payoutAmount = (_insurance.amount * _payoutNumerator) /
          _payoutDenominator;

      vault.borrowValue(_payoutAmount, msg.sender);

      emit Redeemed(
          _id,
          msg.sender,
          _insurance.target,
          _insurance.amount,
          _payoutAmount
      );
  }
```

