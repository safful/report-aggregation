## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Don't assign default values](https://github.com/code-423n4/2022-01-livepeer-findings/issues/215) 

# Handle

0x0x0x


# Vulnerability details

## Concept

When a variable is declared solidity assigns the default value. In case the contract assigns the value again, it costs extra gas. 

Example:`uint x = 0` costs more gas than `uint x` without having any different functionality.

Scope

```
./protocol/bonding/libraries/EarningsPool.sol:84:        uint256 delegatorFees = 0;
./protocol/bonding/libraries/EarningsPool.sol:85:        uint256 transcoderFees = 0;
./protocol/bonding/libraries/EarningsPool.sol:115:        uint256 delegatorRewards = 0;
./protocol/bonding/libraries/EarningsPool.sol:116:        uint256 transcoderRewards = 0;
./protocol/bonding/libraries/EarningsPool.sol:189:        uint256 transcoderFees = 0;
./protocol/bonding/libraries/EarningsPool.sol:190:        uint256 delegatorFees = 0;
./protocol/bonding/libraries/EarningsPool.sol:217:        uint256 transcoderRewards = 0;
./protocol/bonding/libraries/EarningsPool.sol:218:        uint256 delegatorRewards = 0;
./protocol/pm/mixins/MixinTicketBrokerCore.sol:121:        uint256 amountToTransfer = 0;
./protocol/token/Minter.sol:223:        uint256 currentBondingRate = 0;
./arbitrum-lpt-bridge/L1/gateway/L1Migrator.sol:471:        uint256 total = 0;
./protocol/zeppelin/MintableToken.sol:17:    bool public mintingFinished = false;
./protocol/zeppelin/Pausable.sol:13:    bool public paused = false;

```

