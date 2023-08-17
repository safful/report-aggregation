## Tags

- bug
- 3 (High Risk)
- disagree with severity
- sponsor confirmed

# [SpeedBumpPriceGate: Excess ether did not return to the user](https://github.com/code-423n4/2022-05-factorydao-findings/issues/48) 

# Lines of code

https://github.com/code-423n4/2022-05-factorydao/blob/e22a562c01c533b8765229387894cc0cb9bed116/contracts/SpeedBumpPriceGate.sol#L65-L82


# Vulnerability details

## Impact
The passThruGate function of the SpeedBumpPriceGate contract is used to charge NFT purchase fees.
Since the price of NFT will change due to the previous purchase, users are likely to send more ether than the actual purchase price in order to ensure that they can purchase NFT. However, the passThruGate function did not return the excess ether, which would cause asset loss to the user.
Consider the following scenario: 
1. An NFT is sold for 0.15 eth
2. User A believes that the value of the NFT is acceptable within 0.3 eth, considering that someone may buy the NFT before him, so user A transfers 0.3 eth to buy the NFT
3. When user A's transaction is executed, the price of the NFT is 0.15 eth, but since the contract does not return excess eth, user A actually spends 0.3 eth.
## Proof of Concept
https://github.com/code-423n4/2022-05-factorydao/blob/e22a562c01c533b8765229387894cc0cb9bed116/contracts/SpeedBumpPriceGate.sol#L65-L82
## Tools Used
None
## Recommended Mitigation Steps
```
-   function passThruGate(uint index, address) override external payable {
+  function passThruGate(uint index, address payer) override external payable {
        uint price = getCost(index);
        require(msg.value >= price, 'Please send more ETH');

        // bump up the price
        Gate storage gate = gates[index];
        // multiply by the price increase factor
        gate.lastPrice = (price * gate.priceIncreaseFactor) / gate.priceIncreaseDenominator;
        // move up the reference
        gate.lastPurchaseBlock = block.number;

        // pass thru the ether
        if (msg.value > 0) {
            // use .call so we can send to contracts, for example gnosis safe, re-entrance is not a threat here
-           (bool sent, bytes memory data) = gate.beneficiary.call{value: msg.value}("");
+          (bool sent, bytes memory data) = gate.beneficiary.call{value: price}("");
            require(sent, 'ETH transfer failed');
        }
+      if (msg.value - price > 0){ 
+         (bool sent, bytes memory data) = payer.call{value: msg.value - price}("");
+          require(sent, 'ETH transfer failed');}
    }
```

