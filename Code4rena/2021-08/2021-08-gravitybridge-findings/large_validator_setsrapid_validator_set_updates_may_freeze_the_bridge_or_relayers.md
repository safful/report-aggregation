## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Large Validator Sets/Rapid Validator Set Updates May Freeze the Bridge or Relayers](https://github.com/code-423n4/2021-08-gravitybridge-findings/issues/6) 

# Handle

nascent


# Vulnerability details

In a similar vein to "Freeze The Bridge Via Large ERC20 Names/Symbols/Denoms", a sufficiently large validator set or sufficiently rapid validator update could cause both the `eth_oracle_main_loop` and `relayer_main_loop` to fall into a state of perpetual errors. In `find_latest_valset`, [we call](https://github.com/althea-net/cosmos-gravity-bridge/blob/92d0e12cea813305e6472851beeb80bd2eaf858d/orchestrator/relayer/src/find_latest_valset.rs#L33-L40):
```rust=
let mut all_valset_events = web3
            .check_for_events(
                end_search.clone(),
                Some(current_block.clone()),
                vec![gravity_contract_address],
                vec![VALSET_UPDATED_EVENT_SIG],
            )
            .await?;
```

Which if the validator set is sufficiently large, or sufficiently rapidly updated, which continuous return an error if the logs in a 5000 (see: `const BLOCKS_TO_SEARCH: u128 = 5_000u128;`) block range are in excess of 10mb. Cosmos hub says they will be pushing the number of validators up to 300 (currently 125). At 300, each log would produce 19328 bytes of data (4\*32+64\*300). Given this, there must be below 517 updates per 5000 block range otherwise the node will fall out of sync. 

This will freeze the bridge by disallowing attestations to take place.

This requires a patch to reenable the bridge.

## Recommendation
Handle the error more concretely and check if you got a byte limit error. If you did, chunk the search size into 2 and try again. Repeat as necessary, and combine the results.

