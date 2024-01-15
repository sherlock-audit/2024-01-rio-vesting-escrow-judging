Massive Porcelain Albatross

medium

# Incomplete Return Value in OZVotingAdapter

## Summary

The vulnerability involves the lack of propagation of the uint256 return value from the call to `castVote` and `castVoteWithReason` functions in the `OZVotingAdapter` back to the `VestingEscrow` contract.

## Vulnerability Detail

The `vote` and `voteWithReason` act as a wrapper to OZ Governor implementation by doing a delegatecall to OZVotingAdapter from the `VestingEscrow` contract. The functions `castVote` and `castVoteWithReason` return an `uint256` indicating the weight of the voter in the proposal. However this value is not returned to the original caller of the escrow contract nor on the voting adapter itself. This does not follow with the `IGovernor` interface.

## Impact

The inconsistency in return values may lead to integration failures with third-party contracts that depend on accurate and complete data according to the `IGovernor` interface. The inability to obtain the voter's weight in a proposal can affect the functionality and security of systems built upon this contract.

## Code Snippet

 [OZVotingAdaptor.sol#L63-L65](https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/adaptors/OZVotingAdaptor.sol#L63-L65)
[OZVotingAdaptor.sol#L70-L72](https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/adaptors/OZVotingAdaptor.sol#L70-L72)

## Tool used

Manual Review

## Recommendation

Consider modifying OZVotingAdapter's `vote` and `voteWithReason` functions from the `OZVotingAdapter` to return the `uint256` value representing the voter's weight in the proposal. Ensuring the expected behavior of the `IGovernor` interface and enhancing compatibility with other systems. This should make the `VestingEscrow` contract return this value appropriately on the byte stream.
