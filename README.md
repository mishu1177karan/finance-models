# finance-models

Excel-based quantitative finance models built from first principles — each workbook is
self-contained, formula-driven, and readable cell by cell. No macros, no add-ins.

| Model | Topic | File |
|---|---|---|
| Brownian Motion | Simulating a standard Wiener process | `Brownian Motion Excel.xlsx` |
| Vasicek | Short-rate model with mean reversion | `Vasicek interest rate model.xlsx` |
| SA-CCR | Basel counterparty credit risk exposure | `SA-CCR Model.xlsx` |

---

## 1. Brownian Motion

A discrete simulation of standard Brownian motion $W(t)$ over one year, built three times at
different step counts so the effect of discretisation is visible side by side.

**Sheets**

| Sheet | Steps (N) | dt |
|---|---|---|
| `Sheet1` | 252 | 1/252 (daily) |
| `Sheet3` | 250 | 1/250 |
| `Sheet2` | 200 | 1/200 |

**Method**

Each step draws a standard normal shock and scales it by $\sqrt{dt}$:

```
dt = T / N
Z  ~ N(0,1)        =NORMSINV(RAND())
dW = Z * SQRT(dt)
W(t) = W(t-1) + dW    (cumulative)
```

The columns run: `Steps → Time (yrs) → Z~N(0,1) → dW → W(t) cumulative`.

**Properties demonstrated**

1. Brownian motion starts at zero — $W(0) = 0$
2. Increments are independent
3. Increments are normally distributed with variance $dt$

`RAND()` is volatile, so pressing **F9** redraws the whole path — useful for seeing how
different realisations behave under the same parameters.

---

## 2. Vasicek Interest Rate Model

Monte Carlo simulation of the Vasicek short-rate process, with 10 independent paths run
in parallel across columns.

**SDE**

$$dr_t = a(b - r_t)\,dt + \sigma\,dW_t$$

Discretised in the sheet as:

```
r(t+1) = a * (b - r(t)) * dt + σ * SQRT(dt) * NORMSINV(RAND()) + r(t)
```

**Parameters** (cells `B4:B11`, all editable)

| Parameter | Meaning | Value |
|---|---|---|
| `a` | Speed of mean reversion | 0.5 |
| `b` | Long-term mean level | 0.05 |
| `σ` | Volatility | 0.01 |
| `r0` | Current short rate | 0.03 |
| `T` | Horizon (years) | 1 |
| `N` | Steps | 252 |
| Paths | Simulations | 10 |

Starting at 3% with a long-term mean of 5% and a mean-reversion speed of 0.5, the paths
drift upward toward `b` while the diffusion term keeps them noisy. Raising `a` pulls them
in faster; raising `σ` widens the cone.

`Sheet2` holds a separate options-overlay scratchpad (gold implied vol, ADV-based sizing,
put/call/strangle strike selection) — unrelated to the Vasicek simulation.

---

## 3. SA-CCR Model

A full trade-level implementation of the Basel **Standardised Approach for Counterparty
Credit Risk**, following BCBS 279 (March 2014, rev. April 2014).

**Supervisory parameters** (BCBS 279 §183) are held in a lookup table:

| Asset Class | Subclass | SF | ρ | Option Vol |
|---|---|---|---|---|
| Interest Rate | — | 0.5% | — | 50% |
| FX | — | 4.0% | — | 15% |
| Equity | Single Name | 32% | 0.50 | 120% |
| Equity | Index | 20% | 0.80 | 75% |
| Commodity | Electricity | 40% | 0.40 | 150% |
| Commodity | Oil/Gas | 18% | 0.40 | 70% |
| Commodity | Metals | 18% | 0.40 | 70% |

**Sample portfolio** — 12 trades (`T01`–`T12`) spanning vanilla swaps, a cash-settled payer
swaption, an FX forward and FX option, equity forward and index option, commodity forwards
and a sold gold call, plus a SOFR basis swap and an equity variance swap. Notionals are
converted to USD via an FX table.

**Calculation chain**

1. **Trade level** — adjusted notional `d`, supervisory duration `SD`, supervisory delta
   (with `d1` for options), and maturity factor `MF`; effective notional = `d × δ × MF`
2. **Hedging set aggregation**
   - *Interest rate* — bucketed by end date into <1y / 1–5y / >5y, aggregated with the
     prescribed 1.4 / 1.4 / 0.6 correlation weights (§166); basis hedging sets get half SF
   - *FX* — full offset within a currency pair, none across pairs (§170–171)
   - *Equity* — systematic/idiosyncratic decomposition, $\sqrt{(\Sigma\rho A)^2 + \Sigma(1-\rho^2)A^2}$ (§176–177)
   - *Commodity* — same pooling logic by broad category, Energy and Metals kept separate (§178–182)
3. **EAD**

```
PFE AddOn = Σ AddOn(IR, FX, Equity, Commodity)      §150
RC        = max(V − C, 0)
EAD       = 1.4 × (RC + PFE)
```

Volatility and correlation handling for volatility and basis hedging sets is included, so
the workbook covers the special-hedging-set cases as well as the plain vanilla path.

---

## Notes

- Built in Excel with native functions only — `NORMSINV`, `RAND`, `SUMIFS`, `SQRT`.
- The Monte Carlo sheets recalculate on every edit. Switch to
  **Formulas → Calculation Options → Manual** if you want a path to stay put.
- Parameter cells are the intended entry points; formula columns are meant to be traced,
  not overwritten.

## Author

**Mishu Kumar** — [github.com/mishu1177karan](https://github.com/mishu1177karan)
