Main Porcelain Octopus

medium

# [Calculating token is susceptible to precision loss due to division before multiplication](https://github.com/sherlock-audit/2024-01-rio-vesting-escrow-g01u/issues/1)

## Summary
The higher timestamp is divided by a lower timestamp which should argue a precision loss, If the original timestamps have high precision (e.g., nanoseconds), dividing them may lead to a result with lower precision.

## Vulnerability Detail
In `_totalVestedAt` function returns the wrong output,
`return Math.min(_totalLocked * (time - _startTime) / (endTime() - _startTime), _totalLocked);` here,

#PoC
suppose,
`_startTime` is `1000000000000000`.
`time(called)` is `1000000000000100`.
`endTime()` is `1100000000000000`.

hence `(time - _startTime)`  is `100` and 
`endTime() - _startTime` is `100000000000000`
  so,
Math.min(_totalLocked * (100) / (100000000000000), _totalLocked);` 
Math.min(_totalLocked * 0, _totalLocked);` 
Math.min(0, _totalLocked);` //a-b

 the final output is itself `_totalLocked` 



```

 function _totalVestedAt(uint256 time) internal pure returns (uint256) {
        uint40 _startTime = startTime();
        uint256 _totalLocked = totalLocked();
        if (time < _startTime + cliffLength()) {
            return 0;
        }
       @ audit for pricision loss
        return Math.min(_totalLocked * (time - _startTime) / (endTime() - _startTime), _totalLocked); 
    }

```

## Impact
`unclaimed` and `locked` functions are called by `_totalVestedAt`(malicious output) function that differs in the value of token locked and unclaimed making protocol inefficient in calculations.

## Code Snippet
[_totalVestedAt](https://github.com/sherlock-audit/2024-01-rio-vesting-escrow/blob/main/rio-vesting-escrow/src/VestingEscrow.sol#L274)

## Tool used

Manual Review

## Recommendation
return Math.min((_totalLocked * (time - _startTime)) / (endTime() - _startTime), _totalLocked); 
