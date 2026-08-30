# Henry Hub Natural Gas: Forward Curve and Fixed-for-Floating Swap Valuation

Seasonality analysis of the Henry Hub natural gas forward curve and valuation of an
11-month fixed-for-floating commodity swap, built in Python.

This project extends a valuation framework I originally developed for interest rate
swaps on TIIE Fondeo (QuantLib, including CVA/DVA) to a physical commodity underlying.
The mechanics carry over — discount curve, annuity factor, par rate — but the shape of
the forward curve does not, and that difference is the point of the exercise.

---

## Motivation

Interest rate forward curves are driven by monetary policy expectations. Natural gas
forward curves are driven by weather. I wanted to see how far a rates-based valuation
framework transfers to a market where the underlying has to be physically produced,
stored and delivered.

---

## Data

| Series | Source | Notes |
|---|---|---|
| Henry Hub spot price, daily 1997–2026 | EIA, retrieved via FRED (`DHHNGSP`) | USD/MMBtu, not seasonally adjusted |
| Forward curve, 11 contracts | TradingView — NYMEX Henry Hub Natural Gas Futures (root `NG`) | Close of 28 Aug 2026, 14:59 GMT-6 |

**A note on provenance.** The EIA publishes Henry Hub spot prices through at least three
products sourced from three different price reporters, and they disagree on individual
days. "The EIA Henry Hub price" is therefore an incomplete citation without naming the
product. This project uses the FRED-distributed series (`DHHNGSP`) throughout.

The forward curve values are last traded prices from TradingView, **not** official CME
settlement prices. Settlement prices are exchange-calculated (typically a volume-weighted
average over the closing window) and are the correct input for production valuation and
margining. For a curve-shape exercise the difference is immaterial, but it is not zero.

---

## Historical context

![Spot price](output/01_spot_historico.png)

Three episodes dominate the series, and they have structurally different causes:

- **2020 — demand collapse.** COVID-19 lockdowns cut industrial and commercial gas demand.
- **February 2021 — local supply shock.** Winter Storm Uri froze wellheads and gathering
  lines across Texas, cutting production precisely as heating and power-generation demand
  spiked.
- **2022 — global demand shock.** Following the invasion of Ukraine, Europe replaced
  piped Russian gas with seaborne LNG. Because the US is an LNG exporter, European demand
  transmitted into the domestic price — evidence that gas has shifted from a regional to
  a globally linked market.

The distinction matters: one was a demand collapse, one a physical supply constraint, one
a geopolitical demand shift. In commodities, price responds to constraints on production,
storage and transport, not to expectations alone.

---

## Forward curve and seasonality

![Forward curve](output/02_curva_forward.png)

| | Contract | Delivery | USD/MMBtu |
|---|---|---|---|
| Maximum | NGF2027 | Jan 2027 | 3.937 |
| Minimum | NGJ2027 | Apr 2027 | 2.786 |
| **Seasonal amplitude** | | | **1.151 (41.3% above the trough)** |
| Strip average | | Oct 2026 – Aug 2027 | 3.164 |

The curve traces the consumption year rather than a directional view: it climbs into the
January heating peak, falls sharply through March, bottoms in April — after heating
season ends and before cooling demand begins — and recovers into summer on
air-conditioning load.

The steepest single-month move is **February to March: 67 cents**, the market pricing the
end of the heating season.

---

## Swap valuation

An 11-month fixed-for-floating swap on Henry Hub, monthly settlement, 10,000 MMBtu per
month. The floating leg pays the forward price; the problem is to find the fixed price
that makes the swap worth zero at inception.

```
VP(floating) = Σ  F_i · V · df_i
VP(fixed)    = K · V · Σ df_i

K = VP(floating) / (V · Σ df_i)
```

| Result | Value |
|---|---|
| Par fixed price | **3.1651 USD/MMBtu** |
| Simple strip average | 3.1643 USD/MMBtu |
| Difference | +0.08 cents |
| MtM at inception | ~0 (verification) |

**Why the two differ.** The strip average weights every month equally; the swap discounts
each cash flow by its own tenor. The par fixed price is therefore a *discount-weighted*
average of the forward curve, not an arithmetic one. Because the expensive months on this
curve fall early and the cheap months late, discounting penalises the cheap months more
and pulls the fixed price slightly above the strip.

The gap is small here only because the tenor is short: over 11 months at 4%, discount
factors range from roughly 0.99 to 0.96. Raising the rate assumption widens the
difference, and at a zero rate the par price collapses onto the simple strip average —
the arithmetic check that the mechanism is discounting and nothing else:

| Discount rate | Par fixed price | Difference vs strip |
|---|---|---|
| 0% | 3.1643 | +0.00 cents |
| 4% | 3.1651 | +0.08 cents |
| 10% | 3.1662 | +0.19 cents |
| 20% | 3.1679 | +0.37 cents |

---

## Assumptions and limitations

- Flat USD risk-free rate of 4%. A real valuation would use a SOFR discount curve.
- Settlement assumed at month-end of the delivery month.
- The September 2026 contract was excluded: with expiry days away, its price is distorted
  by roll effects. The curve therefore covers 11 contracts, not 12.
- Deterministic valuation. This is appropriate for a plain vanilla swap — the forward
  curve already embeds the market's expectation and risk premium. Stochastic simulation
  would only be required for optionality or for counterparty exposure profiles (CVA).

---

## Repository

```
├── datos/curva_hh.csv        # forward curve, 28 Aug 2026
├── henry_hub_swap.ipynb      # full analysis
└── output/                   # figures
```

**Run:** open the notebook and run all cells. Requires `pandas` and `matplotlib`.

---

## Possible extensions

Adding counterparty credit risk adjustments to this swap, reusing the CDS bootstrapping
and Hull-White exposure simulation from my interest rate swap project — the natural next
step, and the point where deterministic valuation stops being sufficient.

---

*Daniela Ibarra Gatica · [LinkedIn](https://www.linkedin.com/in/daniela-ibarra-gatica-175411338)*
