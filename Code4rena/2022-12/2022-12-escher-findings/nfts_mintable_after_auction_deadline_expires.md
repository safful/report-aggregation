## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-12

# [NFTs mintable after Auction deadline expires](https://github.com/code-423n4/2022-12-escher-findings/issues/474) 

# Lines of code

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L58-L89


# Vulnerability details

## Impact

The `buy` function on the `LPDA.sol` contract is not validating if the auction is still running, allowing a purchase to be made after the stipulated time. The `endtime` variable used to store the end date of the auction is not used at any point to validate whether the purchase is being made within the deadline.

## Proof of Concept

```
// SPDX-License-Identifier: MIT 

pragma solidity ^0.8.17; 

import "forge-std/Test.sol"; 
import {EscherTest} from "./utils/EscherTest.sol"; 
import {LPDAFactory, LPDA} from "src/minters/LPDAFactory.sol"; 

contract LPDABase is EscherTest { 
	LPDAFactory public lpdaSales; 
	LPDA.Sale public lpdaSale; 
	
	function setUp() public virtual override { 
		super.setUp(); 
		lpdaSales = new LPDAFactory();
		// set up a LPDA Sale 
		lpdaSale = LPDA.Sale({ 
			currentId: uint48(0), 
			finalId: uint48(10), 
			edition: address(edition), 
			startPrice: uint80(uint256(1 ether)), 
			finalPrice: uint80(uint256(0.1 ether)), 
			dropPerSecond: uint80(uint256(0.1 ether) / 1 days), 
			startTime: uint96(block.timestamp), 
			saleReceiver: payable(address(69)), 
			endTime: uint96(block.timestamp + 1 days)
		 });
	} 
} 

contract LPDATest is LPDABase { 
	LPDA public sale; 
	
	event End(LPDA.Sale _saleInfo); 
	function test_Buy() public { 
		sale = LPDA(lpdaSales.createLPDASale(lpdaSale)); 
		// authorize the lpda sale to mint tokens
		edition.grantRole(edition.MINTER_ROLE(), address(sale));
		vm.warp(block.timestamp + 3 days); 
		sale.buy{value: 1 ether}(1); 
		assertEq(address(sale).balance, 1 ether); 
	} 
}
```

The code above shows that even after two days after the `endTime` it was still possible to make the purchase.

## Recommended Mitigation Steps

Our recommendation would be to introduce a require to validate the  if the purchase is being made within the `endTime`. 

```
require(block.timestamp > sale.endTime, "TOO LATE");
```

The above could must be placed at the beginning of the `buy` function.