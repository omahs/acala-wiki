# Liquidation

Liquidating unsafe positions requires selling off some collateral assets to repay kUSD in the vault. The liquidation mechanism uses Karura Swap in combination of collateral auction to ensure efficiency and effectiveness. 

The end result of a liquidation is 

* the kUSD debt is repaid
* liquidation fee is collected from the vault owner and added to the `cdp_treasury` as surplus
* remaining collaterals are returned to the vault owner
* in an unfortunate liquidation event where not all kUSD can be recopped, `cdp_treasury` will record the bad debt 

## Liquidate on Karura Swap

It will calculate the target kUSD amount, which is the sum of the kUSD amount owed and the liquidation penalty needs to be paid. It will then attempt to swap collaterals of the vault for the target kUSD amount on Karura Swap if the slippage is within the accepted level. 

If the corresponding kUSD pool on Karura Swap has great liquidity, and the size of the trade is reasonable with regards to the liquidity available, then this will be the most efficient way for liquidaiton. 

Otherwise it will create an auction to auction off the collateral for kUSD. 

## Liquidate on Collateral Auction

Liquidation auction is used to sell off collaterals to recover kUSD and pay back unsafe vaults, if they cannot be liquidated via Karura Swap. A combination of ascending auction and reverse auction mechanism is used to achieve the desirable outcome. Below is a summary of the process:

1. The liquidated collateral will be auctioned in **an ascending auction** \(where bidders bid upwards\) with a preset kUSD target \(outstanding kUSD debt + liquidation penalty\)
2. Then, once such a bid has been reached, the auction switches to **a descending reserve auction** that allows any potential buyers to bid the minimum amount of the collateralized asset they are willing to accept by paying the amount of the preset kUSD target. Auction ends when no lower bid is placed within the auto extension period.
3. Lastly, the part of collateral sold in the auction mechanism is transferred to the auction winner, and any remaining collateral is returned to the original vault owner. The kUSD debt is now repaid, and the liquidation penalty in kUSD is collected by the `cdp-treasury` as surplus.

\[[Source](https://github.com/AcalaNetwork/Acala/blob/master/modules/cdp-engine/src/lib.rs#L372)\]
