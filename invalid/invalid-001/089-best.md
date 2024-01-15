Stable Cornflower Swift

medium

# Anyone can cast a vote through the Voting Adaptor

## Summary
Only escrow recipients and factory owners should be allowed to vote in the `OZVotingAdaptor` contract.

## Vulnerability Detail

There is no access control in the vote function, but we see that in the `vestingEscrow` contract, we want only the recipient of the escrow to call the `vote` function, which is quite unnecessary if it can be circumvented by calling the vote from the adaptor directly.

## Impact

lack of incentive to use the escrow if voting can be done outside an escrow contract without issues

## Code Snippet

https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/adaptors/OZVotingAdaptor.sol#L63

https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/adaptors/OZVotingAdaptor.sol#L70

## Tool used

Manual Review

## Recommendation

Add access control to the `vote` and `voteWithReason` functions
