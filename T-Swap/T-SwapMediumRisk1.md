# Missing `deadline`            

### Relevant GitHub Links

https://github.com/Cyfrin/2024-06-t-swap/blob/main/src/TSwapPool.sol


## Summary
The `deposit` function in the `TSwap` contract contains a vulnerability due to an unused `deadline` parameter. This flaw allows transactions to complete even after the specified `deadline`, which can lead to unexpected behavior and potential exploitation.

## Vulnerability Details
The `deposit` function in the `TSwap` contract has a significant issue where the `deadline `parameter, although defined, is not utilized within the function. This oversight means that the function does not enforce any time constraints, allowing `deposits` to be processed regardless of the specified `deadline`. Users expect that transactions will fail if they do not meet the specified `deadline`, but since this parameter is not checked or used in any way, the function will execute successfully even if the `deadline` has passed. This flaw can lead to unexpected behavior and potential exploitation, as transactions can occur outside the intended time frame. This vulnerability needs to be addressed promptly to ensure the contract operates as intended and maintains its integrity and reliability.
```solidity
function deposit(
        uint256 wethToDeposit,
        uint256 minimumLiquidityTokensToMint,
        uint256 maximumPoolTokensToDeposit,
        uint64 deadline
    )
        external
        revertIfZero(wethToDeposit)
        returns (uint256 liquidityTokensToMint)
    {
        if (wethToDeposit < MINIMUM_WETH_LIQUIDITY) {
            revert TSwapPool__WethDepositAmountTooLow(
                MINIMUM_WETH_LIQUIDITY,
                wethToDeposit
            );
        }
        if (totalLiquidityTokenSupply() > 0) {
            uint256 wethReserves = i_wethToken.balanceOf(address(this));
            uint256 poolTokenReserves = i_poolToken.balanceOf(address(this));
            
            uint256 poolTokensToDeposit = getPoolTokensToDepositBasedOnWeth(
                wethToDeposit
            );
            if (maximumPoolTokensToDeposit < poolTokensToDeposit) {
                revert TSwapPool__MaxPoolTokenDepositTooHigh(
                    maximumPoolTokensToDeposit,
                    poolTokensToDeposit
                );
            }

            // We do the same thing for liquidity tokens. Similar math.
            liquidityTokensToMint =
                (wethToDeposit * totalLiquidityTokenSupply()) /
                wethReserves;
            if (liquidityTokensToMint < minimumLiquidityTokensToMint) {
                revert TSwapPool__MinLiquidityTokensToMintTooLow(
                    minimumLiquidityTokensToMint,
                    liquidityTokensToMint
                );
            }
            _addLiquidityMintAndTransfer(
                wethToDeposit,
                poolTokensToDeposit,
                liquidityTokensToMint
            );
        } else {
            // This will be the "initial" funding of the protocol. We are starting from blank here!
            // We just have them send the tokens in, and we mint liquidity tokens based on the weth
            _addLiquidityMintAndTransfer(
                wethToDeposit,
                maximumPoolTokensToDeposit,
                wethToDeposit
            );
            liquidityTokensToMint = wethToDeposit;
        }
    }
```

## Impact
The unused `deadline` parameter can have several impacts:

1. `Operational Risk`: `Users` and automated systems that depend on the `deadline` for operational correctness may face unexpected behavior, causing potential financial and logistical issues.
2. `Trustworthiness`: The integrity and reliability of the `TSwap` contract may be called into question, leading to a loss of trust among `users` and stakeholders.
3. `Regulatory Complianc`e: For financial contracts, adhering to specific time constraints might be a regulatory requirement. Failing to enforce `deadlines `can lead to non-compliance issues.

## Tools Used
1. Manual Code Review

## Recommendations
Implement the `revertIfDeadlinePassed` `modifier` to enforce the `deadline` check within the `deposit` function and other relevant functions in the contract. This approach ensures a consistent and reusable method for handling `deadlines, reducing the risk of similar issues in the future. 
```diff
function deposit(
        uint256 wethToDeposit,
        uint256 minimumLiquidityTokensToMint,
        uint256 maximumPoolTokensToDeposit,
        uint64 deadline
    )
        external
        revertIfZero(wethToDeposit)
+      revertIfDeadlinePassed(deadline)
        returns (uint256 liquidityTokensToMint)
    {
        if (wethToDeposit < MINIMUM_WETH_LIQUIDITY) {
            revert TSwapPool__WethDepositAmountTooLow(
                MINIMUM_WETH_LIQUIDITY,
                wethToDeposit
            );
        }
        if (totalLiquidityTokenSupply() > 0) {
            uint256 wethReserves = i_wethToken.balanceOf(address(this));
            uint256 poolTokenReserves = i_poolToken.balanceOf(address(this));
           
            uint256 poolTokensToDeposit = getPoolTokensToDepositBasedOnWeth(
                wethToDeposit
            );
            if (maximumPoolTokensToDeposit < poolTokensToDeposit) {
                revert TSwapPool__MaxPoolTokenDepositTooHigh(
                    maximumPoolTokensToDeposit,
                    poolTokensToDeposit
                );
            }

            // We do the same thing for liquidity tokens. Similar math.
            liquidityTokensToMint =
                (wethToDeposit * totalLiquidityTokenSupply()) /
                wethReserves;
            if (liquidityTokensToMint < minimumLiquidityTokensToMint) {
                revert TSwapPool__MinLiquidityTokensToMintTooLow(
                    minimumLiquidityTokensToMint,
                    liquidityTokensToMint
                );
            }
            _addLiquidityMintAndTransfer(
                wethToDeposit,
                poolTokensToDeposit,
                liquidityTokensToMint
            );
        } else {
            // This will be the "initial" funding of the protocol. We are starting from blank here!
            // We just have them send the tokens in, and we mint liquidity tokens based on the weth
            _addLiquidityMintAndTransfer(
                wethToDeposit,
                maximumPoolTokensToDeposit,
                wethToDeposit
            );
            liquidityTokensToMint = wethToDeposit;
        }
    }
```