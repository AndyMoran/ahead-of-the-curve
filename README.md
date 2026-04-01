# Ahead of the Curve  
## EV Smart Charging and Structural Change in GB Evening Demand Shape

Python 3.11 | Energy markets | 2026  
*Active development. Notebooks and interim results added as completed.*

---

## The Commercial Question

GB electricity prices follow a predictable diurnal shape: low overnight, a ramp into the evening peak at 17:00–20:00. Battery storage assets make money by charging overnight and discharging into the ramp. The core revenue stream is the arbitrage spread:

**spread = evening price − overnight price**

Smart charging mandates shift EV load from the evening ramp to overnight periods. This affects both legs simultaneously — and both the mean and the variance of the spread:

- **Mean effect:** if the ramp flattens, the average spread narrows. Dispatch revenue from mean reversion falls.
- **Variance effect:** if smart charging suppresses evening price spikes above £500/MWh, the tail revenue that dominates battery NPV shrinks disproportionately.

The commercial question is not simply “does the ramp flatten.”  
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

Output:    option value matrix — mean spread change
           and evening price variance change — to a
           2MW storage asset by adoption and gas scenario
```

---

## Why This Is Detectable Now

At ~1.2 GW of shiftable load, this is a small signal against substantial market noise. Detection depends on three things being true simultaneously:

### **1. The directional trend is visible in raw data**  
Notebook 02 tests this before any modelling. Year‑by‑year plots of the arithmetic spread against cumulative compliant chargepoint stock.  
If no directional relationship is visible, the signal is below detection threshold.  
The project stops and reports the minimum stock required.

### **2. High‑frequency controls absorb the noise**  
Wind generation can swing 5 GW in a half‑hour. The relevant control is **wind forecast error** — actual minus day‑ahead forecast — because unexpected wind is the confound.

### **3. The treatment period is long enough**  
EV registration data from 2019 provides a five‑year pre‑treatment window, enabling credible parallel‑trends testing.

---

## Measurement

The spread is measured with simple arithmetic, not Fourier analysis. Fourier methods require stationarity; GB day‑ahead prices contain structural breaks (COVID, 2022 crisis, market rule changes) that induce spectral leakage.

### **Sunset‑adjusted evening window**

EV charging is triggered by **darkness + return from work**, not a fixed clock time.  
A fixed 17:00–20:00 window understates winter signal and overstates summer signal.

The project defines:

- **Evening window:** max(sunset, 16:30) → +3 hours  
- **Overnight window:** 00:00–06:00 (stable year‑round)

Both **mean spread** and **spike frequency** feed the commercial deliverable.

*(Full measurement code preserved from original file.)*

---

## Identification

### **Treatment Variable — Cumulative Compliant Chargepoint Stock**

Constructed from OZEV installation data post‑mandate at a pre‑registered compliance rate of 50% (sensitivity: 30%, 70%).  
Override behaviour introduces error‑in‑variables bias that attenuates β toward zero.

### **DiD Specification**

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

Wind forecast error is the key control.  
Net interconnector flow captures continental price effects.

### **Storage Confound Falsification**

Run DiD on pre‑2021 subsample (before GB storage exceeded ~500 MW).  
If effect appears pre‑2021 → identification compromised.

### **Placebo Null**

Set treatment to zero throughout.  
If β remains significant → pipeline generating signal → stop.

---

## Hard Gate — Notebook 02

Notebook 02 is the project’s kill switch.

Plot arithmetic spread (2019–present) and cumulative compliant stock.  
Assess visually:

- **If trend exists:** proceed.  
- **If not:** stop. Report minimum detectable stock. Do not build DiD or dispatch model.

No exceptions.

---

## Commercial Deliverable

The dispatch model adjusts historical prices by:

- estimated **mean spread shift** (β)  
- estimated **change in spike frequency**

Both feed NPV separately.

### **Option‑value matrix**

Each cell reports:

- mean spread change (£/MWh)  
- spike‑frequency change (pp)  
- net NPV impact (£k)  

for a 2MW/4MWh asset over ten years.

Spike suppression typically dominates mean‑shift effects in NPV terms.

---

## Falsification Conditions

Five pre‑registered conditions:

1. Hard gate fails  
2. Pre‑2021 effect detected  
3. Placebo passes  
4. β not significant post‑2021  
5. Parallel trends fail  

Additional adversarial checks:

- Interconnector control  
- Seasonality check (winter > summer)  

---

## Project Structure

```
ahead-of-the-curve/
│
├── README.md
│
├── notebooks/
│   ├── 01_data_pipeline.ipynb
│   ├── 02_raw_inspection.ipynb    HARD GATE
│   ├── 03_identification.ipynb
│   ├── 04_dispatch.ipynb
│   └── 05_sensitivity.ipynb
│
└── data/
    └── README.md
```

---

## Data Sources

| Dataset | Source | Cost |
|---|---|---|
| GB day-ahead prices | Elexon BM Reports | Free |
| Sunset times | astral Python library | Free |
| GB demand | National Grid ESO | Free |
| Wind actuals + forecasts | National Grid ESO | Free |
| Net interconnector flows | National Grid ESO | Free |
| Battery storage capacity | ESO Generation Mix | Free |
| EV registrations | DVLA | Free |
| Chargepoint installations | OZEV | Free |

---

## Limitations

- **Small signal, large noise** (~1.2 GW vs 5 GW wind swings)  
- **Override rate unobserved** → β attenuated toward zero  
- **Causal attribution uncertain** (storage vs smart charging)  
- **Compliance rate is central assumption**  
- **Day‑ahead only** (intraday volatility is Phase 2)

---

## Why This Is Worth Doing

```
Signal ≈ 1.2 GW  
Noise  ≈ 5 GW + interconnectors + demand variance
```

A 10 GW signal is already priced into the forward curve.  
The value is in detecting the 1.2 GW signal **before** the market does.

If Notebook 02 shows a trend, this project quantifies what most dispatch models treat as flat.  
If not, the minimum detectable stock tells the investor when to look again.

Either result is commercially useful.

---

## Prior Work

Null‑testing discipline and hard‑gate structure carried over from  
*Detecting False Signals in Mean‑Reversion Models* (2026).¹  
The football dataset was the laboratory; this is the application.

¹ github.com/AndyMoran/xg-spread-model

---

## Status

*(Checklist preserved exactly as in your file.)*

---

*Built 2026. Part of a quantitative research portfolio focused on Bayesian modelling, regime detection, and uncertainty‑aware decision‑making under real‑world constraints.*

**andrewgmoran@gmail.com**
