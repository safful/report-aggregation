## Tags

- bug
- 3 (High Risk)
- satisfactory
- selected for report
- sponsor confirmed
- H-01

# [Rebalance logic is wrong and this distorts the pool's important states](https://github.com/code-423n4/2023-01-timeswap-findings/issues/269) 

# Lines of code

https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-pool/src/structs/Pool.sol#L679


# Vulnerability details

## Impact
The important states including `long0Balance, long1Balance, long1FeeGrowth, long1ProtocolFees` are wrongly calculated and it breaks the pool's invariant.

## Proof of Concept
The protocol provides a rebalancing functionality and the main logic is implemented in the library `Pool.sol`.
If `param.isLong0ToLong1` is true and the transaction is `TimeswapV2PoolRebalance.GivenLong1`, the protocol calculates the `long1AmountAdjustFees` first and the actual `long0Amount, longFees` and the final `long1Balance` is decided accordingly.
The problem is it is using the wrong parameter `pool.long0Balance` while it is supposed to use `pool.long1Balance` in the line L679.

This leads to wrong state calculation in the following logic. (especially L685 is setting the `long1Balance` to zero).
Furthermore, the protocol is designed as a permission-less one and anyone can call `TimeswapV2Pool.rebalance()`.
An attacker can abuse this to break the pool's invariant and take profit leveraging that.

```solidity
packages\v2-pool\src\structs\Pool.sol
665:     function rebalance(Pool storage pool, TimeswapV2PoolRebalanceParam memory param, uint256 transactionFee, uint256 protocolFee) external returns (uint256 long0Amount, uint256 long1Amount) {
666:         if (pool.liquidity == 0) Error.requireLiquidity();
667:
668:         // No need to update short fee growth.
669:
670:         uint256 longFees;
671:         if (param.isLong0ToLong1) {
672:             if (param.transaction == TimeswapV2PoolRebalance.GivenLong0) {
673:                 (long1Amount, longFees) = ConstantSum.calculateGivenLongIn(param.strike, long0Amount = param.delta, transactionFee, true);
674:
675:                 if (long1Amount == 0) Error.zeroOutput();
676:
677:                 pool.long1Balance -= (long1Amount + longFees);
678:             } else if (param.transaction == TimeswapV2PoolRebalance.GivenLong1) {
    //************************************************************
679:                 uint256 long1AmountAdjustFees = FeeCalculation.removeFees(pool.long0Balance, transactionFee);//@audit-info long0Balance -> long1Balance
    //************************************************************
680:
681:                 if ((long1Amount = param.delta) == long1AmountAdjustFees) {
682:                     long0Amount = ConstantSum.calculateGivenLongOutAlreadyAdjustFees(param.strike, pool.long1Balance, true);
683:
684:                     longFees = pool.long1Balance.unsafeSub(long1Amount);
685:                     pool.long1Balance = 0;
686:                 } else {
687:                     (long0Amount, longFees) = ConstantSum.calculateGivenLongOut(param.strike, long1Amount, transactionFee, true);
688:
689:                     pool.long1Balance -= (long1Amount + longFees);
690:                 }
691:
692:                 if (long0Amount == 0) Error.zeroOutput();
693:             }
694:
695:             pool.long0Balance += long0Amount;
696:
697:             (pool.long1FeeGrowth, pool.long1ProtocolFees) = FeeCalculation.update(pool.liquidity, pool.long1FeeGrowth, pool.long1ProtocolFees, longFees, protocolFee);
698:         } else {
699:             if (param.transaction == TimeswapV2PoolRebalance.GivenLong0) {
700:                 uint256 long0AmountAdjustFees = FeeCalculation.removeFees(pool.long0Balance, transactionFee);//@audit-info
701:
702:                 if ((long0Amount = param.delta) == long0AmountAdjustFees) {
703:                     long1Amount = ConstantSum.calculateGivenLongOutAlreadyAdjustFees(param.strike, pool.long0Balance, false);
704:
705:                     longFees = pool.long0Balance.unsafeSub(long0Amount);
706:                     pool.long0Balance = 0;
707:                 } else {
708:                     (long1Amount, longFees) = ConstantSum.calculateGivenLongOut(param.strike, long0Amount, transactionFee, false);
709:
710:                     pool.long0Balance -= (long0Amount + longFees);
711:                 }
712:
713:                 if (long1Amount == 0) Error.zeroOutput();
714:             } else if (param.transaction == TimeswapV2PoolRebalance.GivenLong1) {
715:                 (long0Amount, longFees) = ConstantSum.calculateGivenLongIn(param.strike, long1Amount = param.delta, transactionFee, false);
716:
717:                 if (long0Amount == 0) Error.zeroOutput();
718:
719:                 pool.long0Balance -= (long0Amount + longFees);
720:             }
721:
722:             pool.long1Balance += long1Amount;
723:
724:             (pool.long0FeeGrowth, pool.long0ProtocolFees) = FeeCalculation.update(pool.liquidity, pool.long0FeeGrowth, pool.long0ProtocolFees, longFees, protocolFee);
725:         }
726:     }
```
## Tools Used
Manual Review

## Recommended Mitigation Steps
Fix the L679 as below.
```solidity
uint256 long1AmountAdjustFees = FeeCalculation.removeFees(pool.long1Balance, transactionFee);
```