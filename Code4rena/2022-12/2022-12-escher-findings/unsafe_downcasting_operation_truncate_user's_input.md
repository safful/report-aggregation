## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- primary issue
- selected for report
- sponsor confirmed
- M-08

# [Unsafe downcasting operation truncate user's input](https://github.com/code-423n4/2022-12-escher-findings/issues/369) 

# Lines of code

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L71
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L82
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L101
https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/FixedPrice.sol#L62


# Vulnerability details

## Impact

Unsafe downcasting operation transcate user's input.

## Proof of Concept

There are a few unsafe downcasting operation that transcate user's input. The impact can be severe or minial.

In FixedPrice.sol,

```solidity
/// @notice buy from a fixed price sale after the sale starts
/// @param _amount the amount of editions to buy
function buy(uint256 _amount) external payable {
	Sale memory sale_ = sale;
	IEscher721 nft = IEscher721(sale_.edition);
	require(block.timestamp >= sale_.startTime, "TOO SOON");
	require(_amount * sale_.price == msg.value, "WRONG PRICE");
	uint48 newId = uint48(_amount) + sale_.currentId;
	require(newId <= sale_.finalId, "TOO MANY");

	for (uint48 x = sale_.currentId + 1; x <= newId; x++) {
		nft.mint(msg.sender, x);
	}

	sale.currentId = newId;

	emit Buy(msg.sender, _amount, msg.value, sale);

	if (newId == sale_.finalId) _end(sale);
}
```

 the amount is unsafely downcasted from uint256 to uint48, note the code:
 
 ```solidity
require(_amount * sale_.price == msg.value, "WRONG PRICE");
uint48 newId = uint48(_amount) + sale_.currentId;
```

the upper limit for uint48 is 281474976710655,

if user wants to buy more than 281474976710655 amount of nft and pay the 281474976710655 * sale price amount, the user can only receive 281474976710655 amount of nft because of the downcasting.

In LPDA.sol, we are unsafely downcasting the price in the buy function

```solidity
receipts[msg.sender].amount += amount;
receipts[msg.sender].balance += uint80(msg.value);

for (uint256 x = temp.currentId + 1; x <= newId; x++) {
	nft.mint(msg.sender, x);
}

sale.currentId = newId;

emit Buy(msg.sender, amount, msg.value, temp);

if (newId == temp.finalId) {
	sale.finalPrice = uint80(price);
	uint256 totalSale = price * amountSold;
	uint256 fee = totalSale / 20;
	ISaleFactory(factory).feeReceiver().transfer(fee);
	temp.saleReceiver.transfer(totalSale - fee);
	_end();
}
```

note the line:

uint80(msg.value) and uint80(price)

In LPDA.sol, same issue exists in the refund function:

```solidity
/// @notice allow a buyer to get a refund on the current price difference
function refund() public {
	Receipt memory r = receipts[msg.sender];
	uint80 price = uint80(getPrice()) * r.amount;
	uint80 owed = r.balance - price;
	require(owed > 0, "NOTHING TO REFUND");
	receipts[msg.sender].balance = price;
	payable(msg.sender).transfer(owed);
}
```

note the downcasting: uint80(getPrice())

this means if the price goes above uint80, it will be wrongly trancated to uint80 for price.

The Downcasting in LPDA.sol is damaging because it tracunated user's fund.

Below is the POC:

add test in LPDA.t.sol

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/test/LPDA.t.sol#L167

```solidity
    function test_LPDA_downcasting_POC() public {

        // make the lpda sales contract
        sale = LPDA(lpdaSales.createLPDASale(lpdaSale));
        // authorize the lpda sale to mint tokens
        edition.grantRole(edition.MINTER_ROLE(), address(sale));
        //lets buy an NFT

        uint256 val = uint256(type(uint80).max) + 10 ether;
        console.log('msg.value');
        console.log(val);
        sale.buy{value: val}(1);
    
    }
```

and import "forge-std/console.sol" in LPDA.sol

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L3

```solidity
import "forge-std/console.sol";
```

and add console.log in LPDA.sol buy function.

https://github.com/code-423n4/2022-12-escher/blob/5d8be6aa0e8634fdb2f328b99076b0d05fefab73/src/minters/LPDA.sol#L69

```solidity
receipts[msg.sender].amount += amount;
receipts[msg.sender].balance += uint80(msg.value);

console.log("truncated value");
console.log(receipts[msg.sender].balance);
```

We run our test:

```solidity
forge test -vv --match test_LPDA_downcasting_POC
```

the output is:

```solidity
Running 1 test for test/LPDA.t.sol:LPDATest
[PASS] test_LPDA_downcasting_POC() (gas: 385619)
Logs:
  msg.value
  1208935819614629174706175
  truncated value
  9999999999999999999

Test result: ok. 1 passed; 0 failed; finished in 3.61ms
```

as we can see, user uses 1208935819614629174706175 to buy in LPDA.sol but the balance is truncated to 9999999999999999999, later user is not able to get the refund they are entitled to because the msg.value is unsafely downcasted.

Also note the downcasting for getPrice in LPDA.sol is also a issue:

the getPrice in LPDA.sol returns a uint256

```solidity
/// @notice the price of the sale
function getPrice() public view returns (uint256) {
	Sale memory temp = sale;
	(uint256 start, uint256 end) = (temp.startTime, temp.endTime);
	if (block.timestamp < start) return type(uint256).max;
	if (temp.currentId == temp.finalId) return temp.finalPrice;

	uint256 timeElapsed = end > block.timestamp ? block.timestamp - start : end - start;
	return temp.startPrice - (temp.dropPerSecond * timeElapsed);
}
```

but this is downcasted into uint80 in function buy and refund.

```solidity
uint80 price = uint80(getPrice()) * r.amount;
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

We recommend the project handle downcasting and use safe casting library to make sure the downcast does not unexpected truncate value.

https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast
