Tiny Velvet Dragonfly

high

# Function check owner in EscrowVesting.sol not functioning as expected

## Summary
The intent of the internal function _checkOwnerOrManager is to check if the msg.sender is either the owner or the manager but instead checks if the msg.sender is the owner and manager 
## Vulnerability Detail
Wrong check or error in code
## Impact
User who is either owner or manager won't be able to call any external function this function is used and any function that uses the modifier checkOwnerOrmanager(), which include revokeUnvested(). 
This will result in Dos for owner or manager
## Code Snippet 
https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrow.sol#L246-L250

checkOwnerOrManager modifier used the _checkOwnerOrManager function 
https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrow.sol#L72-L76

revokeUnvested() uses the checkOwnerOrManager modifier
https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrow.sol#L166-L174

## Tool used
Manual Review

##  Recommendation
&& should be changed to || in the _checkOwnerOrManager function 