## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Extension voting threshold check needs to rerun on each transfer](https://github.com/code-423n4/2021-12-sublime-findings/issues/141) 

# Handle

cmichel


# Vulnerability details

The `Extension` contract correctly reduces votes from the `from` address of a transfer and adds it to the `to` address of the transfer (in case both of them voted on it before), but it does not rerun the voting logic in `voteOnExtension` that actually grants the extension.
This leads to issues where an extension should be granted but is not:

#### POC
- `to` address has 100 tokens and votes for the extension
- `from` address has 100 tokens but does not vote for the extension and transfers the 100 tokens to `to`
- `to` now has 200 tokens, `removeVotes` is run, the `totalExtensionSupport` is increased by 100 to 200. In theory, the threshold is reached and the vote should pass if `to` could call `voteOnExtension` again.
- But their call to `voteOnExtension` with the new balance will fail as they already voted on it (`lastVotedExtension == _extensionVoteEndTime`). The extension is not granted.

## Impact
Extensions that should be granted after a token transfer are not granted.

## Recommended Mitigation Steps
Rerun the threshold logic in `removeVotes` as it has the potential to increase the total support if `to` voted for the extension but `from` did not.


