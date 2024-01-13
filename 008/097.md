Shambolic Pickle Gazelle

medium

# Missing function to cast vote with params

## Summary

The OZVotingAdaptor contract provides functionality to cast votes and to provide a reason when casting a vote, but fails to include the case when the vote should carry parameters. 

## Vulnerability Detail

The [OpenZeppelin Governor interface](https://docs.openzeppelin.com/contracts/4.x/api/governance#IGovernor) provides the following functions to vote:

- `castVote(proposalId, support)`
- `castVoteWithReason(proposalId, support, reason)`
- `castVoteWithReasonAndParams(proposalId, support, reason, params)`

The first two correspond to the functions `vote()` and `voteWithReason()` in the OZVotingAdaptor, but the latter is not present in the implementation.

As a real world example of this use case, the FRAX governance implementation uses this params argument to specify the amount of votes for, against or abstain. See the implementation of `_countVoteFractional()` in the GovernorCountingFractional contract: https://github.com/FraxFinance/frax-governance/blob/e465513ac282aa7bfd6744b3136354fae51fed3c/src/governor/GovernorCountingFractional.sol#L191

```solidity
function _countVoteFractional(
    uint256 proposalId,
    address account,
    uint128 totalWeight,
    bytes memory voteData
) internal {
    require(voteData.length == 48, "GovernorCountingFractional: invalid voteData");

    (uint128 _againstVotes, uint128 _forVotes, uint128 _abstainVotes) = _decodePackedVotes(voteData);
```

## Impact

Medium. The vesting recipient cannot execute a vote that requires parameters if the underlying Governor supports this use case.

## Code Snippet

https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/adaptors/OZVotingAdaptor.sol#L61-L73

## Tool used

Manual Review

## Recommendation

Add a `voteWithReasonAndParams()` function in the OZVotingAdaptor that forwards the call to `castVoteWithReasonAndParams()`. Also ensure to add the function in the VestingEscrow that delegates the implementation to the adaptor.
