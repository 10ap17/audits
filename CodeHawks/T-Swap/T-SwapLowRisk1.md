# Incorrect Parameter Order in `LiquidityAdded Event`            

### Relevant GitHub Links

https://github.com/Cyfrin/2024-06-t-swap/blob/main/src/TSwapPool.sol

## Summary
The `LiquidityAdded` event in the `TSwapPool::_addLiquidityMintAndTransfer` function logs its parameters in an incorrect order. The `poolTokensToDeposit` value should be the third parameter, while the `wethToDeposit` value should be the second. This issue willcause problems for off-chain functions that rely on these events to function correctly.

## Vulnerability Details
In the `_addLiquidityMintAndTransfer` function, the `LiquidityAdded` event is emitted with parameters in an incorrect order. The `poolTokensToDeposit` value, which represents the number of pool tokens being deposited, is currently logged as the second parameter. Conversely, the `wethToDeposit` value, representing the amount of `WETH` being deposited, is logged as the third parameter. This incorrect ordering can lead to misinterpretation of event data by off-chain systems that rely on these events for processing and tracking.

The current implementation of the event emission is as follows:

```solidity
emit LiquidityAdded(msg.sender, poolTokensToDeposit, wethToDeposit);
```

## Impact
1. `Data Misinterpretation`: Off-chain systems that depend on the correct order of event parameters willt misinterpret the data, leading to incorrect processing or malfunction.
2. `Trust Issues`: `Users` and `developers` relying on accurate event data might lose trust in the platform if they encounter inconsistencies.

## Tools Used
1. Manual Code Review

## Recommendations
To fix the parameter order in the `LiquidityAdded` event, the event emission should be corrected as follows:

```diff
- emit LiquidityAdded(msg.sender, poolTokensToDeposit, wethToDeposit);
+ emit LiquidityAdded(msg.sender, wethToDeposit, poolTokensToDeposit);
```
This change ensures that the `wethToDeposit` value is the second parameter and the `poolTokensToDeposit` value is the third parameter, aligning with the expected order.
