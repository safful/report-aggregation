## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-03

# [Users that send funds at a price lower than the current low bid have the funds locked](https://github.com/code-423n4/2022-12-tessera-findings/issues/31) 

# Lines of code

https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L114-L150
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L301-L303


# Vulnerability details

If a user contributes funds after there is no more supply left, and they don't provide a price higher than the current minimum bid, they will be unable to withdraw their funds while the NFT remains unbought.

## Impact
Ether becomes stuck until and unless the NFT is bought, which may never happen

## Proof of Concept
When making a contribution, the user calls the `payable` `contribute()` function. If the supply has already been filled (`fillAtAnyPriceQuantity` is zero), the bid isn't inserted into the queue, so the new bid is not tracked anywhere. When the function reaches `processBidsInQueue()`...:
```solidity
// File: src/modules/GroupBuy.sol : GroupBuy.contribute()   #1

99         function contribute(
100            uint256 _poolId,
101            uint256 _quantity,
102            uint256 _price
103 @>     ) public payable {
104            // Reverts if pool ID is not valid
105            _verifyPool(_poolId);
106            // Reverts if NFT has already been purchased OR termination period has passed
107            (, uint48 totalSupply, , , ) = _verifyUnsuccessfulState(_poolId);
108            // Reverts if ether contribution amount per Rae is less than minimum bid price per Rae
109            if (msg.value < _quantity * minBidPrices[_poolId] || _quantity == 0)
110                revert InvalidContribution();
111            // Reverts if ether payment amount is not equal to total amount being contributed
112            if (msg.value != _quantity * _price) revert InvalidPayment();
113    
114            // Updates user and pool contribution amounts
115            userContributions[_poolId][msg.sender] += msg.value;
116            totalContributions[_poolId] += msg.value;
117    
118            // Calculates remaining supply based on total possible supply and current filled quantity amount
119            uint256 remainingSupply = totalSupply - filledQuantities[_poolId];
120            // Calculates quantity amount being filled at any price
121            uint256 fillAtAnyPriceQuantity = remainingSupply < _quantity ? remainingSupply : _quantity;
122    
123            // Checks if quantity amount being filled is greater than 0
124 @>         if (fillAtAnyPriceQuantity > 0) {
125                // Inserts bid into end of queue
126                bidPriorityQueues[_poolId].insert(msg.sender, _price, fillAtAnyPriceQuantity);
127                // Increments total amount of filled quantities
128                filledQuantities[_poolId] += fillAtAnyPriceQuantity;
129            }
130    
131            // Calculates unfilled quantity amount based on desired quantity and actual filled quantity amount
132            uint256 unfilledQuantity = _quantity - fillAtAnyPriceQuantity;
133            // Processes bids in queue to recalculate unfilled quantity amount
134 @>         unfilledQuantity = processBidsInQueue(_poolId, unfilledQuantity, _price);
135    
136            // Recalculates filled quantity amount based on updated unfilled quantity amount
137            uint256 filledQuantity = _quantity - unfilledQuantity;
138            // Updates minimum reserve price if filled quantity amount is greater than 0
139            if (filledQuantity > 0) minReservePrices[_poolId] = getMinPrice(_poolId);
140    
141            // Emits event for contributing ether to pool based on desired quantity amount and price per Rae
142            emit Contribute(
143                _poolId,
144                msg.sender,
145                msg.value,
146                _quantity,
147                _price,
148                minReservePrices[_poolId]
149            );
150:       }
```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L99-L150

...if the price isn't higher than the lowest bid, the while loop is broken out of, with `pendingBalances` having never been updated, and the function does not revert:
```solidity
// File: src/modules/GroupBuy.sol : GroupBuy.processBidsInQueue()   #2

291        function processBidsInQueue(
292            uint256 _poolId,
293            uint256 _quantity,
294            uint256 _price
295        ) private returns (uint256 quantity) {
296            quantity = _quantity;
297            while (quantity > 0) {
298                // Retrieves lowest bid in queue
299                Bid storage lowestBid = bidPriorityQueues[_poolId].getMin();
300                // Breaks out of while loop if given price is less than than lowest bid price
301 @>             if (_price < lowestBid.price) {
302 @>                 break;
303 @>             }
304    
305                uint256 lowestBidQuantity = lowestBid.quantity;
306                // Checks if lowest bid quantity amount is greater than given quantity amount
307                if (lowestBidQuantity > quantity) {
308                    // Decrements given quantity amount from lowest bid quantity
309                    lowestBid.quantity -= quantity;
310                    // Calculates partial contribution of bid by quantity amount and price
311                    uint256 contribution = quantity * lowestBid.price;
312    
313:                   // Decrements partial contribution amount of lowest bid from total and user contributions
```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L291-L313

In order for a user to get funds back, the amount must have been stored in `pendingBalances`, and since this is never done, all funds contributed during the `contribute()` call become property of the `GroupBuy` contract, with the user being unable to withdraw...:
```solidity
// File: src/modules/GroupBuy.sol : GroupBuy.withdrawBalance()   #3

274        function withdrawBalance() public {
275            // Reverts if caller balance is insufficient
276 @>         uint256 balance = pendingBalances[msg.sender];
277 @>         if (balance == 0) revert InsufficientBalance();
278    
279            // Resets pending balance amount
280            delete pendingBalances[msg.sender];
281    
282            // Transfers pending ether balance to caller
283            payable(msg.sender).call{value: balance}("");
284:       }
```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L274-L284

...until the order has gone through, and they can `claim()` excess funds, but there likely won't be any, due to the separate MEV bug I raised:
```solidity
// File: src/modules/GroupBuy.sol : GroupBuy.contribution   #4

228        function claim(uint256 _poolId, bytes32[] calldata _mintProof) external {
229            // Reverts if pool ID is not valid
230            _verifyPool(_poolId);
231            // Reverts if purchase has not been made AND termination period has not passed
232            (, , , bool success, ) = _verifySuccessfulState(_poolId);
233            // Reverts if contribution balance of user is insufficient
234 @>         uint256 contribution = userContributions[_poolId][msg.sender];
235            if (contribution == 0) revert InsufficientBalance();
236    
237            // Deletes user contribution from storage
238            delete userContributions[_poolId][msg.sender];
```
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L228-L244


## Tools Used
Code inspection

## Recommended Mitigation Steps
`revert()` if the price is lower than the min bid, and the queue is already full
