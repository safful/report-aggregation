## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Freeze Bridge via Non-UTF8 Token Name/Symbol/Denom](https://github.com/code-423n4/2021-08-gravitybridge-findings/issues/4) 

# Handle

nascent


# Vulnerability details

Manual insertion of non-utf8 characters in a token name will break parsing of logs and will always result in the oracle getting in a loop of failing and early returning an error. The fix is non-trivial and likely requires significant redesign.

# Proof of Concept
Note the `c0` in the last argument of the call data (invalid UTF8).

It can be triggered with:
```solidity=
data memory bytes = hex"f7955637000000000000000000000000000000000000000000000000000000000000008000000000000000000000000000000000000000000000000000000000000000c000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000000000000000000000000012000000000000000000000000000000000000000000000000000000000000000461746f6d0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000046e616d6500000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000673796d626fc00000000000000000000000000000000000000000000000000000";
gravity.call(data);
```
The log output is as follows:
```
ERC20DeployedEvent("atom", "name", ❮utf8 decode failed❯: 0x73796d626fc0, 18, 2)
```

Which hits [this code path](https://github.com/althea-net/cosmos-gravity-bridge/blob/92d0e12cea813305e6472851beeb80bd2eaf858d/orchestrator/gravity_utils/src/types/ethereum_events.rs#L431-L438):
```rust=
            let symbol = String::from_utf8(input.data[index_start..index_end].to_vec());
            trace!("Symbol {:?}", symbol);
            if symbol.is_err() {
                return Err(GravityError::InvalidEventLogError(format!(
                    "{:?} is not valid utf8, probably incorrect parsing",
                    symbol
                )));
            }
```

And would cause an early return [here](https://github.com/althea-net/cosmos-gravity-bridge/blob/92d0e12cea813305e6472851beeb80bd2eaf858d/orchestrator/orchestrator/src/ethereum_event_watcher.rs#L99):
```rust=
let erc20_deploys = Erc20DeployedEvent::from_logs(&deploys)?;
```

Never updating last checked block and therefore, this will freeze the bridge by disallowing any attestations to take place. This is an extremely low cost way to bring down the network.

## Recommendation
This is a hard one. Resyncing is permanently borked because on the Go side, there is seemingly no way to ever process the event nonce because protobufs do not handle non-utf8 strings. The validator would report they need event nonce `N` from the orchestrator, but they can never parse the event `N`. Seemingly, validators & orchestrators would have to know to ignore that specific event nonce. But it is a permissionless function, so it can be used to effectively permanently stop attestations & the bridge until a new `Gravity.sol` is deployed.

One potential fix is to check in the solidity contract if the name contains valid utf8 strings for denom, symbol and name. This likely will be expensive though. Alternatively, you could require that validators sign ERC20 creation requests and perform checks before the transaction is sent.


