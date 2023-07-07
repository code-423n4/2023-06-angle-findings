## 1. UNUSED VARIABLES CAN BE OMITTED TO SAVE GAS

In the `LibOracle.getBurnOracle()` function, declares the `uint256 oracleValueTmp` memory variable and later initializes it as follows by calling the `readBurn` function:

        (oracleValueTmp, ratioObserved) = readBurn(ts.collaterals[collateralList[i]].oracleConfig);

But the `oracleValueTmp` variable is never used within the scope of the function and is used only as a place holder. Hene we can omit this variable and save extra `MSTORE` for each iteration of the `for` loop. Hence the gas saved will be equal to `3 * collateralList.length`; As a result if the list of collateral assets supported by this system is higher this will save considerable amount of gas.

The line of code can be edited as follows:

        ( , ratioObserved) = readBurn(ts.collaterals[collateralList[i]].oracleConfig);

https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/libraries/LibOracle.sol#L79-L84

## 2. `storage` VARIABLE CAN BE REPLACED WITH `calldata` VARIABLE TO SAVE GAS 

In the `Swapper._buildPermitTransferPayload` the `Collateral` struct is passed into the function as a `storage` variable named `collatInfo`. But it is only read from and not written to within the scope of the function implementation. Hence it is recommended to make the `collatInfo` a `calldata` variable and read from `calldata` instead, to save on two extra `SLOAD` operations for the transaction.

https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/facets/Swapper.sol#L497
https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/facets/Swapper.sol#L500

## 3. REDUNDANT CHECKS CAN BE OMITTED TO SAVE GAS

In the `Swapper.swapExactOutputWithPermit()` function performs a check to verify that the `amountIn` is less than the `amountInMax` amount as follows:

        if (amountIn > amountInMax) revert TooBigAmountIn(); 

But this is a redundant check since the same check is performed as shown below, in the `SignatureTransfer.sol` contract which is a parent contract of `Permit2.sol` contract. This is called inside the `Swapper._swap()` function.

        if (requestedAmount > permit.permitted.amount) revert InvalidAmount(permit.permitted.amount);

Hence the redundant call inside the `Swapper.swapExactOutputWithPermit()` function can be omitted to save gas.

https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/facets/Swapper.sol#L139

## 4. CONDITIONAL CHECK CAN BE PERFORMED DURING VARIABLE INITIALIZATION TO SAVE GAS

In the `DistributionCreator._createDistribution()` function the `userFeeRebate` value is used when calculating the `distributionAmountMinusFees` if the `UniswapV3Pool` tokens are not whitelisted. Before the `distributionAmountMinusFees` calculatation, there is a check inside the `if` statement as follows:

    userFeeRebate < BASE_9

This check makes sure that the `distributionAmountMinusFees` will only be calculated if the `userFeeRebate < BASE_9`. 

But it is recommended to check whether `userFeeRebate < BASE_9` in the `DistributionCreator.setUserFeeRebate` function when setting the `feeRebate[user] = userFeeRebate`. By doing this we can omit the `userFeeRebate < BASE_9` check in the `DistributionCreator._createDistribution()` and also we can prevent setting the `feeRebate[user]` value as invalid ( > BASE_9) value thus saving a `SSTORE` as well.

The suggested code snippet change is as follows:

    function setUserFeeRebate(address user, uint256 userFeeRebate) external onlyGovernorOrGuardian {
        require(userFeeRebate < BASE_9, "Invalid userFeeRebate");
        feeRebate[user] = userFeeRebate;
        emit FeeRebateUpdated(user, userFeeRebate);
    }

https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/DistributionCreator.sol#L392-L395
https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/DistributionCreator.sol#L240

## 5. `uint256` VARIABLE INITIALIZATION TO DEFAULT VALUE OF `0` CAN BE OMMITTED

There is no need to initialize variables to their default values during declaration, since they are any way initialized to default value once declared.

https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/transmuter/facets/Redeemer.sol#L199


## 6. ARITHMATIC OPERATIONS CAN BE UNCHECKED, SINCE THERE IS NO UNDERFLOW OR OVERFLOW

In the `DistributionCreator.toggleTokenWhitelist` function implementation can be `unchecked` to save gas.

This function is used to toggle the value of `isWhitelistedToken[token]` between `0` adn `1`. Hence the arithematic operation can never underflow or overflow. Hence the `DistributionCreator.toggleTokenWhitelist` function can be updated as follows with the `unchecked` keyword.

    function toggleTokenWhitelist(address token) external onlyGovernorOrGuardian {
        unchecked{
            uint256 toggleStatus = 1 - isWhitelistedToken[token];
            isWhitelistedToken[token] = toggleStatus;
        }
        emit TokenWhitelistToggled(token, toggleStatus);
    } 

https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/DistributionCreator.sol#L398-L402
https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/Distributor.sol#L266-L267
https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/Distributor.sol#L273-L274
https://github.com/AngleProtocol/merkl-contracts/blob/1825925daef8b22d9d6c0a2bc7aab3309342e786/contracts/Distributor.sol#L208-L209
https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/savings/Savings.sol#L221-L223

In the `SavingsVest.accrue()` function, when the protocol is under-collateralized and the `missing <= currentLockedProfit` the vesting profit updated as follows:

            } else {
                vestingProfit = currentLockedProfit - missing;
                lastUpdate = uint64(block.timestamp);
            }

This arithmetic operation can be unchecked due to previous check of `if (missing > currentLockedProfit)`.

Hence the above code snippet can be modified as follows:

            } else {
                unchecked{
                    vestingProfit = currentLockedProfit - missing;
                    lastUpdate = uint64(block.timestamp);
                }
            }

https://github.com/AngleProtocol/angle-transmuter/blob/9707ee4ed3d221e02dcfcd2ebaa4b4d38d280936/contracts/savings/SavingsVest.sol#L134-L137
