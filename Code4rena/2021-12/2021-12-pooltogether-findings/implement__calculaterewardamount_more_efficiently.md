## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Implement _calculateRewardAmount more efficiently](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/134) 

# Handle

0x0x0x


# Vulnerability details

[https://github.com/pooltogether/v4-periphery/blob/b520faea26bcf60371012f6cb246aa149abd3c7d/contracts/TwabRewards.sol#L302-L321](https://github.com/pooltogether/v4-periphery/blob/b520faea26bcf60371012f6cb246aa149abd3c7d/contracts/TwabRewards.sol#L302-L321) is as follows:

```
        uint256 _averageBalance = _ticket.getAverageBalanceBetween(
            _user,
            uint64(_epochStartTimestamp),
            uint64(_epochEndTimestamp)
        );

        uint64[] memory _epochStartTimestamps = new uint64[](1);
        _epochStartTimestamps[0] = uint64(_epochStartTimestamp);

        uint64[] memory _epochEndTimestamps = new uint64[](1);
        _epochEndTimestamps[0] = uint64(_epochEndTimestamp);

        uint256[] memory _averageTotalSupplies = _ticket.getAverageTotalSuppliesBetween(
            _epochStartTimestamps,
            _epochEndTimestamps
        );

        if (_averageTotalSupplies[0] > 0) {
            return (_promotion.tokensPerEpoch * _averageBalance) / _averageTotalSupplies[0];
        }

        return 0;
    }
```

Since `_averageBalance` is always bigger than `_averageTotalSupplies[0]`. We can implement the, if statement earlier. This will ensure to output 0 earlier. Furthermore, `_averageBalance` is in stack and this check costs less gas. Therefore, the code can be implemented as follows:

```
        uint256 _averageBalance = _ticket.getAverageBalanceBetween(
            _user,
            uint64(_epochStartTimestamp),
            uint64(_epochEndTimestamp)
        );
				if (_averageBalance > 0) {
		        uint64[] memory _epochStartTimestamps = new uint64[](1);
		        _epochStartTimestamps[0] = uint64(_epochStartTimestamp);
		
		        uint64[] memory _epochEndTimestamps = new uint64[](1);
		        _epochEndTimestamps[0] = uint64(_epochEndTimestamp);
		
		        uint256[] memory _averageTotalSupplies = _ticket.getAverageTotalSuppliesBetween(
		            _epochStartTimestamps,
		            _epochEndTimestamps
		        );

            return (_promotion.tokensPerEpoch * _averageBalance) / _averageTotalSupplies[0];
        }

        return 0;
    }
```

