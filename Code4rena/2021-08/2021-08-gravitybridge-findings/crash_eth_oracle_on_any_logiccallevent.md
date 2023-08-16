## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Crash Eth Oracle On Any LogicCallEvent](https://github.com/code-423n4/2021-08-gravitybridge-findings/issues/11) 

# Handle

nascent


# Vulnerability details

**Severity: Medium**
**Likelihood: High**

In `eth_oracle_main_loop`, `get_last_checked_block` is called. Followed by:
```rust=
let logic_call_executed_events = web3
            .check_for_events(
                end_search.clone(),
                Some(current_block.clone()),
                vec![gravity_contract_address],
                vec![LOGIC_CALL_EVENT_SIG],
            )
            .await;
```
and may hit the code path:
```rust=
        for event in logic_call_executed_events {
            match LogicCallExecutedEvent::from_log(&event) {
                Ok(call) => {
                    trace!(
                        "{} LogicCall event nonce {} last event nonce",
                        call.event_nonce,
                        last_event_nonce
                    );
                    if upcast(call.event_nonce) == last_event_nonce && event.block_number.is_some()
                    {
                        return event.block_number.unwrap();
                    }
                }
                Err(e) => error!("Got ERC20Deployed event that we can't parse {}", e),
            }
        }
```

But will panic at `from_log` here:
```rust=
impl LogicCallExecutedEvent {
    pub fn from_log(_input: &Log) -> Result<LogicCallExecutedEvent, GravityError> {
        unimplemented!()
    }
    // snip...
}
```
It can/will also be triggered here in `check_for_events`: 
```rust=
let logic_calls = LogicCallExecutedEvent::from_logs(&logic_calls)?;
```

Attestations will be frozen until patched.

## Recommendation
Implement the method.

## Recommended Mitigation Steps

