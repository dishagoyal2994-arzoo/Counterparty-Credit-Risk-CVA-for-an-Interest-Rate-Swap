# Counterparty Credit Risk: CVA for an Interest Rate Swap

A from-scratch Monte Carlo model for pricing counterparty credit risk (CVA) on a
plain-vanilla interest rate swap — the "will the other side of my trade still be
solvent" question that sits at the intersection of derivative pricing and credit risk.

Fully simulation-based; no external dataset required. Calibrated to one real market
input (current SOFR) with all other parameters explicitly labeled as illustrative.

## Key Results
- Par swap rate at inception: **3.785%** (swap value at inception: $0.00, confirming internal consistency)
- Peak Expected Exposure: **$345,876** at t=4.92y; Peak 95% PFE: **$571,852** at t=1.92y
- CVA (typical 150bps counterparty spread): **$11,234** (0.112% of $10M notional)
- Stress test: CVA ranges from **$3,902** (high-grade, 50bps) to **$27,085** (stressed,
  400bps) — roughly a **6.9x** difference purely from counterparty credit quality

## What's in the Notebook
1. Simulate 5,000 future short-rate paths using a Vasicek mean-reverting process
2. Price zero-coupon bonds analytically under Vasicek (closed-form, no extra
   simulation layer needed for discount factors)
3. Set up a 5-year receive-fixed interest rate swap, decomposed as fixed-leg-value
   minus floating-leg-value, and solve for the par swap rate at inception
4. Build the counterparty exposure profile: Expected Exposure (EE) and 95%
   Potential Future Exposure (PFE), taking only the positive side of swap value
   (default only costs you money if the swap is in your favor)
5. Convert the exposure profile into a single CVA figure using a constant hazard-
   rate default assumption
6. Stress-test CVA against counterparty credit quality (50bps / 150bps / 400bps)

## Note on the Two Vasicek Models
This project and the credit risk project both use something called "the Vasicek
model," but for different risk factors: here it describes the short-term interest
rate's evolution; in the credit project, a related single-factor Vasicek framework
describes correlated borrower defaults. Same mathematical family (mean-reverting
stochastic process), different application.

## Assumptions & Limitations
- Only `r0` (today's short rate) is a live market input; mean-reversion speed,
  volatility, and long-run mean are illustrative, not fitted to a historical series
  or full yield curve
- The floating leg uses the standard "resets to par" shortcut rather than
  simulating each individual reset — this is why Expected Exposure rises toward
  maturity rather than showing the more typical humped shape of a production model
  (the 95% PFE, by contrast, does peak earlier, around year 2 — tail scenarios can
  spike before the average risk builds up)
- The counterparty's hazard rate is backed out from one flat illustrative spread,
  not bootstrapped from an actual CDS term structure
- No netting, collateral/CSA, or wrong-way risk is modeled — all standard next
  steps for a production CVA engine

## Tech Stack
Python · NumPy · pandas · matplotlib

## How to Run
No external data required. Install dependencies, then run top to bottom. Before
using results anywhere formal, check the current SOFR print at
fred.stlouisfed.org/series/SOFR and update `r0`.
