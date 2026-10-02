# Fixed Income Analytics: Yield-Curve PCA and Factor Hedging

Solo project for the Rutgers MQF Fixed Income Securities course (Python, Jupyter). It reprices a real Treasury bond portfolio off an interpolated curve, computes its key-rate delta ladder, decomposes curve moves with PCA, hedges the dominant factors with zero-coupon bonds, and backtests the hedge against the realized next-day curve move.

## Files
- `PCA_Hedge_Fixed_Income_Analytics.ipynb` (72 cells) and a rendered `.pdf`.
- `TLT_holdings_10272025.csv`: iShares TLT (20+ Year Treasury Bond ETF) holdings, 2025-10-27.
- `Yield_Curve.xlsx`: US Treasury par-yield curve, 14 pillars from 1 month to 30 years, about 214 daily rows.

## What it does
1. **Data ingestion:** parses the TLT holdings file, keeps the 44 Treasury bonds (fixed income asset class with a valid CUSIP), and cleans comma and percent formatted fields.
2. **Curve construction:** 14 par-yield pillars turned into a continuous `y(t)` by linear interpolation and by cubic spline; discount factors `DF(t) = exp(-y(t) t)`.
3. **Pricing:** semiannual coupon schedules generated backwards from maturity, price as discounted cash flows, portfolio PV aggregated. Model portfolio PV is about 50.76 billion dollars as of 2025-10-27.
4. **Key-rate delta ladder:** bump each of the 14 pillars by 1 bp, rebuild the curve, fully reprice, and take `delta_i = (PV_bumped - PV_base) / dy_i`.
5. **Yield-curve PCA:** on daily yield changes from 2025-09-01 to 2025-10-27 (33 changes), eigen-decomposition of the 14 by 14 covariance matrix. PC1 is level (all loadings positive); PC2 is slope (sign change near 2Y).
6. **Closed-form ZCB ladder:** a T-year zero is sensitive only to its two bracketing pillars, `dP/dy0 = -T P (1 - alpha)` and `dP/dy1 = -T P alpha`.
7. **PC1 hedge:** portfolio PC1 exposure is `delta_ladder @ PC1`; solve for the 10Y zero-coupon notional that neutralizes it.
8. **Backtest (10/27 to 10/28):** first-order `PnL = sum_i delta_i dy_i` using actual next-day yield changes. Unhedged P&L of 166.1 million dollars falls to 30.5 million after the PC1 hedge, about 82% of the move removed.
9. **PC1 + PC2 hedge:** a 2 by 2 linear system gives 10Y and 2Y zero notionals that neutralize both factors.

## Limitations
Par yields are interpolated directly rather than bootstrapped to zero rates; the PCA window is short (33 observations); the backtest is a single day and first order only (no convexity); PC1 and PC2 explained-variance percentages are not reported.

## Next steps
OIS or dual-curve bootstrap, PC3 (curvature) hedge with a third instrument, second-order P&L, and a rolling hedge-effectiveness backtest.
