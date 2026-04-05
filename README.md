# Ahead of the Curve  
## EV Smart Charging and Structural Change in GB Evening Demand Shape

Python 3.11 | Energy markets | 2026  

---

## Findings — Notebook 02 Gate: FAILED

> **The signal is not yet detectable. The project stops at Notebook 02.**  
> This is a pre-registered outcome. The hard gate did exactly what it was designed to do.

### What was tested

GB electricity prices follow a diurnal shape: low overnight, elevated during the evening peak at roughly 16:00–19:00. Battery storage assets monetise the spread between these windows. Smart charging mandates shift EV load from evening into overnight periods, which should compress this spread.

The question was: **is a directional signal already visible in raw GB price data?**

### What was found

The evening spread has been falling since 2022, and cumulative compliant chargepoint stock has been rising — producing a raw Spearman correlation of **ρ = −0.407 (p = 0.004)**. This looks promising. But after controlling for wind forecast error (actual minus day-ahead forecast), the signal disappears entirely:

| Test | ρ | p |
|---|---|---|
| Spread ~ compliant stock (raw) | −0.407 | 0.004 |
| Spread ~ compliant stock (wind-controlled) | −0.084 | 0.575 |

Wind forecast error explains **31.8%** of monthly spread variance. Once removed, there is no residual chargepoint signal. The raw correlation is a confound: declining spreads and rising EV stock both happened to trend in the same post-2022 period, for unrelated reasons.

### Why the signal is undetectable

The arithmetic is unfavourable at current penetration:

```
Shiftable load  ≈  490,575 chargepoints × 7 kW × 30% simultaneity
                =  1.03 GW

Wind noise (SD) ≈  1,436 MW forecast error per half-hour
                →  daily average noise ≈ comparable to signal
```

A 1.03 GW signal against ~1.4 GW of wind noise gives a signal-to-noise ratio below detection threshold. The spread distribution is also dominated by gas price variance and interconnector flows — further burying the smart charging effect.

### Minimum detectable stock

To produce a wind-controlled partial correlation of ρ = −0.25 (a weak but genuine signal) would require approximately:

| Parameter | Value |
|---|---|
| Required compliant chargepoints | ~12,000,000 |
| Required shiftable load | ~25 GW |
| Current compliant chargepoints | 490,575 |
| Current shiftable load | 1.03 GW |
| Multiple required | ~24× current |

At central BEV adoption forecasts this threshold is reached around **2032–2035**, when GB BEV stock exceeds approximately 8 million vehicles. The signal may become detectable earlier if:
- Simultaneity rates increase (managed charging shifts load more predictably)
- Gas price volatility falls (reducing the dominant noise source)
- Additional flexibility mandates raise compliance rates above 50%

### What this means commercially

This null result is itself commercially useful. It tells a storage asset operator that:

1. The smart charging effect on evening spread is **not yet priced into the forward curve** — no adjustment is needed to dispatch models today
2. The effect will become material — and potentially mispriced — in the early 2030s
3. The **correct time to re-run this analysis** is when GB BEV stock approaches 4–5 million vehicles (roughly 2028–2030), giving lead time before the signal reaches the market

Either result from a hard gate — pass or fail — was pre-registered as commercially useful. A failed gate with a quantified revisit threshold is more useful than a spurious positive.

---

## The Commercial Question

GB electricity prices follow a predictable diurnal shape: low overnight, a ramp into the evening peak at 17:00–20:00. Battery storage assets make money by charging overnight and discharging into the ramp. The core revenue stream is the arbitrage spread:

**spread = evening price − overnight price**

Smart charging mandates shift EV load from the evening ramp to overnight periods. This affects both legs simultaneously — and both the mean and the variance of the spread:

- **Mean effect:** if the ramp flattens, the average spread narrows. Dispatch revenue from mean reversion falls.
- **Variance effect:** if smart charging suppresses evening price spikes above £500/MWh, the tail revenue that dominates battery NPV shrinks disproportionately.

The commercial question is not simply "does the ramp flatten."  
It is: **what happens to the full distribution of the spread, and what is that worth to a 2MW storage asset?**

This project estimates both the mean shift and the variance change and reports the option value without assuming the sign.

---

## TL;DR

```python
# Pre-registered prior: expected shiftable load at current penetration
evs             = 1.1e6  # UK pure EVs, DVLA early 2026
charger_power   = 7      # kW, average home charger
compliance_rate = 0.50   # smart charging mandate take-up
simultaneity    = 0.30   # fraction charging during evening window

shiftable_gw = (evs * charger_power * compliance_rate * simultaneity) / 1e6
# ≈ 1.155 GW — roughly 2% of GB evening peak demand
#
# SNR note: GB wind can swing 5 GW in a single half-hour.
# This is a small signal in a noisy market.
# Detection relies on high-frequency controls and
# a directional visual signal in raw data.
# If Notebook 02 shows no trend: project stops.
```

```
Signal:    change in mean and variance of evening spread
Method:    arithmetic spread + DiD with wind forecast
           error, demand, and gas price controls
Treatment: cumulative compliant chargepoint stock
           (continuous), instrumented by mandate date
Data:      N2EX/EPEX day-ahead prices 2019–present
           National Grid ESO demand, wind actuals,
           wind day-ahead forecasts, storage capacity
           OZEV chargepoint installation data
           DVLA EV registrations (quarterly)

Hard gate: Notebook 02. No directional trend in raw
           spread vs cumulative EV stock → project stops.
           No exceptions.

Output:    Gate FAILED. Minimum detectable stock
           reported. Revisit threshold: ~2032–2035.
```

---

## Why This Was Worth Doing

```
Signal ≈ 1.2 GW  
Noise  ≈ 5 GW + interconnectors + demand variance
```

A 10 GW signal is already priced into the forward curve.  
The value is in detecting the 1.2 GW signal **before** the market does.

Notebook 02 confirms the signal is not yet detectable. That confirmation is itself information: the spread distribution does not yet reflect smart charging, and dispatch models do not need adjustment today. The quantified revisit threshold — ~12M compliant chargepoints, ~2032–2035 — tells the investor when to look again.

Either result is commercially useful. A hard gate that fails is not a failed project.

---

## Why This Is Detectable (Eventually)

At ~1.2 GW of shiftable load, this is a small signal against substantial market noise. Detection depends on three things being true simultaneously:

**1. The directional trend is visible in raw data**  
Notebook 02 tests this before any modelling. Year‑by‑year plots of the arithmetic spread against cumulative compliant chargepoint stock. At current penetration, the trend is not visible once wind is controlled for.

**2. High‑frequency controls absorb the noise**  
Wind generation can swing 5 GW in a half‑hour. The relevant control is wind forecast error — actual minus day‑ahead forecast — because unexpected wind is the confound. This control was applied; it absorbed the apparent signal.

**3. The treatment period is long enough**  
EV registration data from 2019 provides a five‑year pre‑treatment window. The post-mandate period (June 2022–present) is less than four years — likely insufficient for a 1 GW signal at current SNR.

---

## Measurement

The spread is measured with simple arithmetic, not Fourier analysis. Fourier methods require stationarity; GB day‑ahead prices contain structural breaks (COVID, 2022 crisis, market rule changes) that induce spectral leakage.

### Sunset‑adjusted evening window

EV charging is triggered by darkness + return from work, not a fixed clock time.  
A fixed 17:00–20:00 window understates winter signal and overstates summer signal.

- **Evening window:** max(sunset, 16:30) → +3 hours  
- **Overnight window:** 00:00–06:00 (stable year‑round)

Both mean spread and spike frequency were computed. Neither produced a wind-controlled signal.

---

## Identification (Not Run)

Notebook 03 (DiD specification) was not run. The hard gate in Notebook 02 prevents execution of downstream notebooks. The DiD specification is preserved in the codebase for future use when the signal becomes detectable.

The pre-registered specification was:

```
spread(t) = α
          + β × compliant_stock(t)
          + γ × wind_forecast_error(t)
          + δ × demand(t)
          + ε × gas_price(t)
          + ζ × net_interconnector_flow(t)
          + season_fe
          + u(t)
```

---

## Falsification Conditions

Five pre‑registered conditions. Only Condition 1 was evaluated:

| Condition | Status | Result |
|---|---|---|
| 1. Hard gate passes | **EVALUATED** | **FAILED — project stopped** |
| 2. Pre-2021 effect absent | Not run | — |
| 3. Placebo null holds | Not run | — |
| 4. β significant post-2021 | Not run | — |
| 5. Parallel trends hold | Not run | — |

---

## Data Sources

| Dataset | Source | Status |
|---|---|---|
| GB day-ahead prices | Elexon Insights API (MID) | ✓ Fetched |
| Sunset times | astral Python library | ✓ Computed |
| GB demand | NESO Historic Demand Data | ✓ Fetched |
| Wind actuals | NESO Historic Generation Mix | ✓ Fetched |
| Wind DA forecasts | NESO Day Ahead Wind Forecast | ✓ Fetched |
| Net interconnector flows | NESO Historic Demand Data | ✓ Fetched |
| Battery storage capacity | NESO Historic Generation Mix | ✓ Fetched |
| EV registrations | DVLA VEH0141 | ✓ Fetched |
| Chargepoint installations | DVLA-derived (OZEV discontinued) | ✓ Constructed |

**Note on data sources:** The NESO API domain migrated from `api.nationalgrideso.com` to `api.neso.energy` in 2024. Historic demand data is published as one CSV per year (not a single paginated resource). The old Elexon BMRS API (`api.bmreports.com`) was decommissioned in May 2024; all data now comes from the Insights API (`data.elexon.co.uk`). The OZEV home chargepoint grant scheme statistics were discontinued; compliant stock is constructed from DVLA BEV registration data with a 50% compliance rate assumption.

---

## Project Structure

```
ahead-of-the-curve/
│
├── README.md
│
├── notebooks/
│   ├── 01_data_pipeline.ipynb     ✓ Complete
│   ├── 02_raw_inspection.ipynb    ✓ Complete — GATE FAILED
│   ├── 03_identification.ipynb    — Not run (gate enforcement)
│   ├── 04_dispatch.ipynb          — Not run
│   └── 05_sensitivity.ipynb       — Not run
│
└── data/
    ├── prices_raw.parquet         127,155 half-hours (2019–2026)
    ├── demand_raw.parquet          73,536 half-hours (2021–2026)
    ├── wind_total_raw.parquet     127,160 half-hours (2019–2026)
    ├── interconnector_raw.parquet 126,192 half-hours (2019–2026)
    └── chargepoints.parquet           133 months (2014–2025)
```

---

## Limitations

- **Small signal, large noise** (~1.03 GW vs ~1.4 GW wind forecast error SD)
- **Override rate unobserved** — drivers who override smart charging introduce error-in-variables bias
- **Compliance rate assumed** — 50% central case; actual rate unobserved
- **Post-mandate period short** — less than four years of treatment data
- **Embedded wind only in NESO demand data** — transmission wind required separate source

---

## Prior Work

Null‑testing discipline and hard‑gate structure carried over from  
*Detecting False Signals in Mean‑Reversion Models* (2026).¹  
The football dataset was the laboratory; this is the application.  
The gate failed there too. Both failures are informative.

¹ github.com/AndyMoran/xg-spread-model

---

*Built 2026. Part of a quantitative research portfolio focused on Bayesian modelling, regime detection, and uncertainty‑aware decision‑making under real‑world constraints.*

**andrewgmoran@gmail.com**

