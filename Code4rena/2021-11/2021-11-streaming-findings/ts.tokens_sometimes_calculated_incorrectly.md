## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [ts.tokens sometimes calculated incorrectly](https://github.com/code-423n4/2021-11-streaming-findings/issues/123) 

# Handle

gpersoon


# Vulnerability details

## Impact
Suppose someone stakes some tokens and then withdraws all of his tokens (he can still withdraw). This will result in ts.tokens being 0.

Now after some time he stakes some tokens again.
At the second stake updateStream() is called and the following if condition is false because ts.tokens==0
```JS
  if (acctTimeDelta > 0 && ts.tokens > 0) {
```
Thus ts.lastUpdate is not updated and stays at the value from the first withdraw.
Now he does a second withdraw. updateStream() is called an calculates the updated value of ts.tokens.
However it uses ts.lastUpdate, which is the time from the first withdraw and not from the second stake. So the value of ts.token is calculated incorrectly.
Thus more tokens can be withdrawn than you are supposed to be able to withdraw.

## Proof of Concept
https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L417-L447

```JS
function stake(uint112 amount) public lock updateStream(msg.sender) {
      ...         
        uint112 trueDepositAmt = uint112(newBal - prevBal);
       ... 
        ts.tokens += trueDepositAmt;
```

https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L455-L479

```JS
function withdraw(uint112 amount) public lock updateStream(msg.sender) {
        ...
        ts.tokens -= amount;
```

https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L203-L250

```JS
function updateStreamInternal(address who) internal {
...
 uint32 acctTimeDelta = uint32(block.timestamp) - ts.lastUpdate;
            if (acctTimeDelta > 0 && ts.tokens > 0) {
                // some time has passed since this user last interacted
                // update ts not yet streamed
                ts.tokens -= uint112(acctTimeDelta * ts.tokens / (endStream - ts.lastUpdate));
                ts.lastUpdate = uint32(block.timestamp);
            }
```

## Tools Used

## Recommended Mitigation Steps
Change the code in updateStream()  to:

```JS
    if (acctTimeDelta > 0 ) {
                // some time has passed since this user last interacted
                // update ts not yet streamed
                if (ts.tokens > 0) 
                      ts.tokens -= uint112(acctTimeDelta * ts.tokens / (endStream - ts.lastUpdate));
                ts.lastUpdate = uint32(block.timestamp);  // always update ts.lastUpdate (if time has elapsed)
            }
```

Note: the next if statement with unstreamed and lastUpdate can be changed in a similar way to save some gas


