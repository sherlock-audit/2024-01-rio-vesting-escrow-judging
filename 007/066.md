Clever Brick Okapi

medium

# Block Re-org attack in VestingEscrowFactory

## Summary
Blockchain re-orgs can occur as seen here: https://decrypt.co/101390/ethereum-beacon-chain-blockchain-reorg. Therefore, it is not uncommon as someone might thing. 

With this issue being present, it is important to note that the `VestingEscrowFactory` contract  uses the `clone` functionas depicted in the `deployVestingContract` function here https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrowFactory.sol#L51-L75 from @solady is defined using the `create` function as seen here: https://github.com/Vectorized/solady/blob/77809c18e010b914dde9518956a4ae7cb507d383/src/utils/LibClone.sol#L106

This introduces a possibility of where a malicious users can create a `vestingEscrow` contract clone with their own parameters allowing them to steal funds sent to that address. 

As seen in this previous bug report https://github.com/code-423n4/2023-04-frankencoin-findings/issues/155 and here https://code4rena.com/reports/2023-01-rabbithole#m-01-questfactory-is-suspicious-of-the-reorg-attack the project can consider using `create2` function with a `salt` that includes `msg.sender`. 
## Vulnerability Detail
The following lines of code contains the above described vulnerability https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrowFactory.sol#L51-L75
## Impact
This could lead to lose of vested tokens 
## Code Snippet
https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrowFactory.sol#L51-L75
## Tool used

Manual Review

## Recommendation
As indicated earlier the project can consider using `create2` function with salt. 