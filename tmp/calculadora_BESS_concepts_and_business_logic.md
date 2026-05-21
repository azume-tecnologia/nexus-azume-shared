# Calculadora BESS — Concepts & Business Logic

Plain-text capture of the business logic encoded in `docs/blueprints/sheets/calculadora_BESS.xlsx`. This document is the source of truth for any future re-implementation (API, frontend calculator, spec, or audit). Cell references like `Premissas!C8` are kept inline so a reader can trace back to the workbook.

---

## 1. Purpose & domain context

### What the calculator solves

Brazilian industrial and commercial customers connected at medium voltage (**Grupo A**) pay two very different prices for electricity within the same day:

- **Tarifa Ponta (peak)** — applied during a ~3-hour window on business days. Typical value used in the workbook: **R$ 3.89/kWh**.
- **Tarifa Fora Ponta (off-peak)** — all other hours. Typical value: **R$ 0.60/kWh**.

The peak/off-peak ratio is ~6.5x. Any technology that lets the customer **avoid drawing energy from the grid during the peak window** has very high commercial value. The calculator quantifies four such strategies (the "load shift" play) over a 12-year horizon.

### The four scenarios

| Code | Strategy | Energy source during peak |
|---|---|---|
| **C1 — BESS Puro** | Battery charges off-peak from the grid, discharges during peak | Stored grid energy bought cheap |
| **C2 — BESS + Solar (sized for BESS)** | Solar PV sized just to cover the battery's recharge | Stored solar energy |
| **C3 — BESS + Solar (sized for BESS + off-peak)** | Solar PV sized to also offset the customer's off-peak consumption | Stored solar + solar offset on off-peak bill |
| **C4 — Gerador Diesel** | Diesel genset runs during peak hours instead of drawing from grid | Diesel fuel |

C4 is the **legacy benchmark** — many industrial sites already own a genset for outages, so it's the comparator the customer mentally anchors on.

### Workbook layout

| Tab | Role |
|---|---|
| `Premissas` | All inputs (yellow) and globally-derived values (green) |
| `C1 - BESS Puro` | Scenario 1 dimensioning + monthly/annual economics + CAPEX + simple & escalated payback + 12-year cumulative |
| `C2 - BESS+Solar BESS` | Scenario 2 (same structure as C1 + solar cascade block) |
| `C3 - BESS+Solar FP` | Scenario 3 (same structure as C2, bigger PV array; cascade also tracks unmet off-peak) |
| `C4 - Gerador Diesel` | Scenario 4 (diesel sizing + fuel cost + payback + 12-year cumulative) |
| `Comparativo` | One-page side-by-side of all four scenarios (pure cross-tab aggregation) |
| `Fluxo de Caixa` | 13-row cash flow per scenario (year 0 through year 12) → NPV, IRR. The escalated-payback formulas live on each scenario tab but read the cumulative columns here |

Cell colour code (per `Premissas!B2`): **yellow = input · green = calculated · salmon = diesel-specific**.

---

## 2. Inputs

All cells listed here live in `Premissas`. Defaults shown are the values currently in the workbook. Each row carries:

- **Kind** — `input` (yellow cell in the workbook, user-edited) or `calculated` (green cell, derived from other cells).
- **Formula** — empty for inputs; symbolic expression plus the raw Excel form for calculated cells. `IFERROR(…, 0)` wrappers are stripped for readability (the workbook applies them on every formula; see §7 caveat #20).

### Tariffs

| Input | Cell | Unit | Default | Kind | Formula | Role |
|---|---|---|---|---|---|---|
| Tarifa fora ponta | `C4` | R$/kWh | 0.60 | input | — | Cost to recharge BESS from grid; basis for solar abatement |
| Tarifa ponta | `C5` | R$/kWh | 3.89 | input | — | What every kWh of avoided peak consumption is worth |

### Consumption profile

| Input | Cell | Unit | Default | Kind | Formula | Role |
|---|---|---|---|---|---|---|
| Consumo na ponta | `C8` | kWh/mês | 4,500 | input | — | Energy that BESS/genset must deliver each month |
| Consumo fora ponta | `C9` | kWh/mês | 30,000 | input | — | Baseline off-peak bill; ceiling for solar abatement in C2/C3 |
| Consumo p/ carregar BESS | `C10` | kWh/mês | 4,932 | calculated | `consumo_ponta / η_round_trip (=C8/C20)` | Grid kWh needed each month to deliver `C8` after round-trip losses |
| Consumo total mensal | `C11` | kWh/mês | 34,932 | calculated | `consumo_fora_ponta + consumo_p/_carregar_BESS (=C9+C10)` | Reference total kWh the site pulls from grid (displayed on Premissas; not consumed downstream) |
| Dias úteis com ponta por mês | `C12` | dias | 21 | input | — | Days the peak window is active |
| Horas de ponta por dia | `C13` | h/dia | 3 | input | — | Length of the peak window — drives inverter and genset kW |

### BESS technical parameters

| Input | Cell | Unit | Default | Kind | Formula | Role |
|---|---|---|---|---|---|---|
| Profundidade de descarga (DoD) | `C16` | % | 90% | input | — | Usable fraction of nominal kWh (LFP-typical) |
| η carga AC→DC | `C17` | % | 96% | input | — | Inverter efficiency on the way in |
| η bateria LFP | `C18` | % | 99% | input | — | Internal cycle losses |
| η descarga DC→AC | `C19` | % | 96% | input | — | Inverter efficiency on the way out |
| η round-trip | `C20` | % | 91.2% | calculated | `η_carga · η_bateria · η_descarga (=C17·C18·C19)` | Lumped AC→AC efficiency; charges all losses against the recharge bill |
| Degradação anual da bateria | `C21` | %/ano | 2% | input | — | Linear annual reduction of usable savings |
| Vida útil para análise | `C22` | anos | 12 | input | — | Horizon for cash flow / NPV / IRR |

### Solar PV — inputs (used by C2 & C3)

| Input | Cell | Unit | Default | Kind | Formula | Role |
|---|---|---|---|---|---|---|
| Geração estimada por kWp | `C25` | kWh/kWp/mês | 120 | input | — | Brazilian reference: 100–130 |
| Custo solar instalado | `C26` | R$/kWp | 2,000 | input | — | C&I 2025 reference: R$ 1,800–2,500/kWp |

### Solar PV — sizing C2 (covers BESS recharge only)

| Input | Cell | Unit | Default | Kind | Formula | Role |
|---|---|---|---|---|---|---|
| kWp sugerido C2 | `C29` | kWp | 41.1 | calculated | `consumo_p/_carregar_BESS / geração_por_kWp (=C10/C25)` | PV size that just covers the monthly BESS recharge |
| kWp instalado C2 | `C30` | kWp | 42 | input | — | Manual rounded-up override of `C29`; drives CAPEX and monthly generation |
| Geração mensal C2 | `C31` | kWh/mês | 5,040 | calculated | `kWp_instalado_C2 · geração_por_kWp (=C30·C25)` | Monthly PV output feeding the C2 cascade |

### Solar PV — sizing C3 (covers BESS recharge + full off-peak)

| Input | Cell | Unit | Default | Kind | Formula | Role |
|---|---|---|---|---|---|---|
| kWp sugerido C3 | `C34` | kWp | 291.1 | calculated | `(consumo_p/_carregar_BESS + consumo_fora_ponta) / geração_por_kWp (=(C10+C9)/C25)` | PV size that covers BESS recharge **and** full off-peak consumption |
| kWp instalado C3 | `C35` | kWp | 292 | input | — | Manual rounded-up override of `C34`; drives CAPEX and monthly generation |
| Geração mensal C3 | `C36` | kWh/mês | 35,040 | calculated | `kWp_instalado_C3 · geração_por_kWp (=C35·C25)` | Monthly PV output feeding the C3 cascade |

### Financial parameters

| Input | Cell | Unit | Default | Kind | Formula | Role |
|---|---|---|---|---|---|---|
| CAPEX BESS instalado por kWh | `C39` | R$/kWh | 1,800 | input | — | Multiplies BESS capacity → BESS CAPEX |
| O&M anual (% do CAPEX total) | `C40` | %/ano | 0.5% | input | — | Flat annual maintenance cost |
| Taxa de desconto (TMA) | `C41` | %/ano | 12% | input | — | Discount rate for NPV |
| Reajuste tarifário anual | `C42` | %/ano | 7% | input | — | Tariff escalation applied to annual savings |

### Diesel parameters (C4 only)

| Input | Cell | Unit | Default | Kind | Formula | Role |
|---|---|---|---|---|---|---|
| CAPEX gerador | `C45` | R$ | 220,000 | input | — | Equipment + install + electrical panel + ART |
| Fator de potência cos φ | `C46` | % | 80% | input | — | To convert kW → kVA for sizing |
| Margem de segurança | `C47` | % | 20% | input | — | Headroom for motor starts |
| Consumo específico | `C48` | L/kWh | 0.28 | input | — | Modern gensets: 0.25–0.30 |
| Preço do diesel | `C49` | R$/L | 7.30 | input | — | Local market price |
| Manutenção anual | `C50` | R$/ano | 10,000 | input | — | Oil/filters/inspections (typical 3–7% of CAPEX) |
| Reajuste do diesel | `C51` | %/ano | 7% | input | — | Independent from tariff escalation |

### Diesel sizing (C4 only)

| Input | Cell | Unit | Default | Kind | Formula | Role |
|---|---|---|---|---|---|---|
| Potência gerador | `C53` | kW | 71.4 | calculated | `consumo_ponta / dias_úteis / horas_ponta (=C8/C12/C13)` | Active power the genset must deliver during the peak window |
| Potência aparente | `C54` | kVA | 107.1 | calculated | `(kW_gerador / cos_φ) · (1 + margem_segurança) (=C53/C46·(1+C47))` | Nameplate sizing including power-factor conversion and safety headroom |

---

## 3. Derived global values (Premissas)

These are computed once on `Premissas` and reused across every scenario.

### Round-trip efficiency

```
η_round_trip = η_carga · η_bateria · η_descarga         (Premissas!C20)
             = 0.96 · 0.99 · 0.96 ≈ 91.2%
```

### Energy needed to charge the BESS

```
consumo_para_carregar_BESS = consumo_ponta / η_round_trip   (Premissas!C10)
                           = 4,500 / 0.9124 ≈ 4,932 kWh/mês
```

Higher than `consumo_ponta` because the system loses ~9% in the round trip.

### Solar sizing — suggested vs installed

C2 covers **only** the BESS recharge:
```
kWp_sugerido_C2 = consumo_para_carregar_BESS / geração_por_kWp     (C29)
                = 4,932 / 120 ≈ 41.1 kWp
kWp_instalado_C2 = 42  (rounded up, C30 — used for CAPEX & generation)
```

C3 covers BESS recharge **plus** the customer's full off-peak consumption:
```
kWp_sugerido_C3 = (consumo_para_carregar_BESS + consumo_fora_ponta) / geração_por_kWp   (C34)
                = (4,932 + 30,000) / 120 ≈ 291.1 kWp
kWp_instalado_C3 = 292  (rounded up, C35)
```

The **installed** value is what feeds CAPEX and monthly generation — never the suggested one.

### Diesel sizing

```
kW_gerador = consumo_ponta / dias_úteis / horas_ponta              (C53)
           = 4,500 / 21 / 3 ≈ 71.4 kW
kVA_gerador = (kW / cos_φ) · (1 + margem_segurança)                (C54)
            = (71.4 / 0.8) · 1.2 ≈ 107.1 kVA
```

Same critério as the BESS inverter, but **without** the DoD divisor — a genset delivers instantaneously, no storage losses.

---

## 4. Scenarios

All four scenarios share the same BESS sizing (when they have a BESS). What differs is the energy mix and the savings model.

### Shared BESS sizing (C1, C2, C3)

```
consumo_diário_ponta = consumo_ponta / dias_úteis                          (Cn!C4)
                     = 4,500 / 21 ≈ 214.3 kWh/dia

capacidade_BESS_kWh = consumo_diário_ponta / (η_descarga · DoD)            (Cn!C5)
                    = 214.3 / (0.96 · 0.9) ≈ 248 kWh

potência_inversor_kW = capacidade_BESS_kWh / horas_ponta                   (Cn!C6)
                     = 248 / 3 ≈ 82.7 kW
```

Note: the two BESS formulas split the efficiencies cleanly at the AC boundary.
- **Capacity sizing** (`capacidade_BESS_kWh`) uses only **discharge eff (`C19`)** and **DoD (`C16`)** — these define how much *nameplate* kWh the battery must hold to deliver the daily AC peak load. Charge and battery-internal losses don't change the *storage* requirement; they change the *grid input* requirement.
- **Recharge energy** (`consumo_para_carregar_BESS = C8 / C20`) uses the full **round-trip eff** `C20 = C17·C18·C19`, so all three losses are charged against the recharge bill. `C19` therefore appears in *both* formulas, but in different roles: as a DC→AC conversion factor for capacity, and as part of the lumped AC→AC round-trip for recharge.

### C1 — BESS Puro

**Energy flow:** charge BESS off-peak from grid → discharge during peak.

**Monthly cash:**
```
receita_ponta_evitada = consumo_ponta · tarifa_ponta                       (C1!C9)
                      = 4,500 · 3.89 = R$ 17,505/mês

custo_recarga_rede = consumo_para_carregar_BESS · tarifa_fora_ponta        (C1!C10)
                   = 4,932 · 0.60 ≈ R$ 2,959/mês

economia_mensal = receita_ponta_evitada − custo_recarga_rede               (C1!C11)
                ≈ R$ 14,546/mês

economia_anual_bruta = economia_mensal · 12                                (C1!C12)
                     ≈ R$ 174,549/ano
```

**CAPEX & O&M:**
```
CAPEX_BESS = capacidade_BESS_kWh · CAPEX_por_kWh                           (C1!C15)
           = 248 · 1,800 ≈ R$ 446,429

O&M_anual = CAPEX_BESS · 0.5%                                              (C1!C16)
          ≈ R$ 2,232/ano

economia_anual_líquida = economia_anual_bruta − O&M_anual                  (C1!C17)
                       ≈ R$ 172,316/ano
```

### C2 — BESS + Solar para carregar o BESS

**Solar sized just to cover the BESS recharge.** Anything left over is small (~108 kWh/mês) and offsets a sliver of the off-peak bill.

**Cascade logic** (energy is allocated in order, no export to grid):

```
cobertura_recarga = min(geração_solar, consumo_para_carregar_BESS)         (C2!C12)
                  = min(5,040, 4,932) = 4,932 kWh/mês  ← solar fully covers

saldo_solar = max(0, geração_solar − consumo_para_carregar_BESS)           (C2!C13)
            = max(0, 5,040 − 4,932) = 108 kWh/mês

abatimento_FP = min(saldo_solar, consumo_fora_ponta)                       (C2!C14)
              = min(108, 30,000) = 108 kWh/mês  ← surplus offsets off-peak

recarga_residual = max(0, consumo_para_carregar_BESS − geração_solar)      (C2!C15)
                 = 0  ← solar covered everything
```

**Monthly cash:**
```
economia_mensal = receita_ponta_evitada
                − (recarga_residual · tarifa_fora_ponta)
                + (abatimento_FP · tarifa_fora_ponta)                      (C2!C21)
```

If solar exceeds the BESS need, `recarga_residual = 0` and the off-peak savings show up as a small positive bonus.

**CAPEX:** `CAPEX_BESS + (kWp_instalado_C2 · custo_solar)` → R$ 446,429 + R$ 84,000 = R$ 530,429.

### C3 — BESS + Solar para BESS e Fora Ponta

**Same cascade as C2**, with one extra display-only step and a ~7x larger solar array. With 35,040 kWh/mês of generation vs only 4,932 kWh/mês needed to recharge the BESS, ~30,108 kWh/mês is available to offset the entire 30,000 kWh/mês off-peak consumption.

The extra cell `C3!C15` ("Consumo FP remanescente pago" = `MAX(0, consumo_fora_ponta − abatimento_FP)`, equals 0 with current defaults) is a display-only row that shows the residual off-peak energy the customer would still pay for if solar can't fully cover it. It is *not* read by any cash formula — `abatimento_FP` (`C3!C14`) and `recarga_residual` (`C3!C16`) are the cells that feed the monthly economics. A backend can compute it for parity with the workbook UI but doesn't need it for the cash flow.

**Monthly cash:**
```
economia_mensal = receita_ponta_evitada
                − (recarga_residual · tarifa_fora_ponta)
                + (abatimento_FP · tarifa_fora_ponta)                      (C3!C22)
                ≈ 17,505 + 18,000 = R$ 35,505/mês
```

Effectively the customer pays only the minimal contracted demand and taxes on off-peak — virtually the entire variable bill is wiped out.

**CAPEX:** `CAPEX_BESS + (kWp_instalado_C3 · custo_solar)` → R$ 446,429 + R$ 584,000 ≈ R$ 1,030,429.

### C4 — Gerador Diesel

**Energy flow:** during peak hours, the genset starts and supplies the load directly. No grid draw, no battery, no solar.

**Monthly fuel:**
```
horas_operação_mês = horas_ponta · dias_úteis             = 63 h/mês     (C4!C6)
consumo_diesel_h   = kW_gerador · consumo_específico      = 20.0 L/h     (C4!C7)
consumo_diesel_mês = consumo_diesel_h · horas_operação    = 1,260 L      (C4!C8)
custo_combustível  = consumo_diesel_mês · preço_diesel    ≈ R$ 9,198/mês (C4!C9)
```

**Monthly cash:**
```
economia_mensal = receita_ponta_evitada − custo_combustível              (C4!C14)
                = 17,505 − 9,198 ≈ R$ 8,307/mês
```

**Annual baselines** (separate from the BESS scenarios because diesel uses a different escalator in the cash-flow model):
```
receita_ponta_base  = R$ 210,060/ano   (C4!C15)  ← grows by reajuste_tarifário
custo_diesel_base   = R$ 110,376/ano   (C4!C16)  ← grows by reajuste_diesel
```

---

## 5. Financial model — 12-year cash flow

`Fluxo de Caixa` builds a 13-row (year 0 through year 12) table per scenario. Two columns per scenario: liquid year cash flow and cumulative.

> **Terminology gotcha — "Economia líquida anual" is used twice.** The workbook applies the label `Economia líquida anual` to *two different cells* in each BESS scenario: `Cn!C12/C22/C23` (peak revenue minus variable cost, **before** O&M) and `Cn!C17/C29/C30` (same minus annual O&M). The cash-flow formulas use the *pre-O&M* version and subtract a fixed O&M outside the compounded expression. This doc calls the pre-O&M figure `economia_anual_bruta` for clarity, but a reader looking at the workbook will see "líquida" on both. Don't conflate the two.

### Year 0

```
Year_0 = −CAPEX_total
```

### Years 1–12 (BESS scenarios C1, C2, C3)

```
liquid_n = economia_anual_bruta
         · (1 − degradação_bateria)^(n−1)
         · (1 + reajuste_tarifário)^(n−1)
         − O&M_anual
```

Two compounding effects fight each other: tariffs grow (boosts savings), the battery delivers slightly less every year (shrinks savings). At 7% tariff escalation vs 2% degradation, the **net effect is growth** — savings increase over the life.

> **Exponent is `n−1`, not `n`.** Year 1 has *no* escalation and *no* degradation (both factors are `^0 = 1`). Year 12 has 11 years of each. A backend that uses `^n` instead of `^(n−1)` will overshoot NPV/IRR materially.

### Years 1–12 (diesel C4)

```
liquid_n = receita_ponta_base · (1 + reajuste_tarifário)^(n−1)
         − custo_diesel_base · (1 + reajuste_diesel)^(n−1)
         − manutenção_anual
```

Diesel is modelled with **independent escalators** for the energy it displaces (tariff) and the fuel it consumes (diesel price). Historically, diesel in Brazil has outpaced electricity inflation — at parity (both 7% in defaults), C4 grows steadily; if diesel escalates faster, C4 deteriorates.

### Cumulative

```
cumulative_n = cumulative_{n−1} + liquid_n
```

### NPV (Valor Presente Líquido)

```
NPV = NPV(TMA, liquid_1..liquid_12) + liquid_0      (Fluxo de Caixa!B18, D18, F18, H18)
```

Year 0 is added back outside the `NPV()` call because Excel's `NPV()` treats its first argument as occurring **at end of period 1**, not at t=0 — the convention is to compute NPV of years 1+ then add the un-discounted year-0 outflow.

### IRR (Taxa Interna de Retorno)

```
IRR = IRR(liquid_0..liquid_12)                       (Fluxo de Caixa!B19, D19, F19, H19)
```

Direct Excel IRR on the full 13-element vector; wrapped in `IFERROR(…, "—")` for cases where no real root exists.

### Simple payback (no escalation)

```
ratio = CAPEX_total / economia_anual_líquida
payback = INT(ratio) & " anos e " & ROUND((ratio − INT(ratio))·12, 0) & " meses"
```

Displayed as "X anos e Y meses". Falls back to "Não paga no período".

### Payback with tariff escalation + battery degradation (workbook label: "Payback com reajuste tarifário")

> **Not a discounted payback.** Despite being the more honest metric vs the simple one, this is *not* discounted by TMA — the workbook scans the **nominal** cumulative (escalated + degraded but undiscounted) for the sign change. A true discounted payback would discount each year by `(1+TMA)^n` before accumulating, which would produce a longer payback. If a backend needs a strict discounted payback, compute it separately.

The formula uses a `MATCH(0, cumulative_range, 1)` trick:

1. `MATCH(0, cum, 1)` — with the cumulative cash flow monotonically increasing from negative to positive, `MATCH(…, 1)` returns the position of the **last value ≤ 0** (i.e. last negative year). **Precondition:** the cumulative must be monotonically ascending; if any year's flow is negative enough to send the cumulative back down, `MATCH` mis-reports.
2. The fractional crossover year is computed by linear interpolation between that last negative cumulative and the next (positive) cumulative.
3. The fractional part is converted to months and stitched into the "X anos e Y meses" string.

Wrapped in `IFERROR(…, "Não paga no período")` for scenarios that stay negative throughout the 12-year horizon.

---

## 6. Outputs

### `Comparativo` tab

Reads only from the other tabs (no calculations of its own; pure aggregation) and presents:

- **Sistema** — capacity (kWh), inverter/genset power (kW), generator kVA, installed kWp.
- **CAPEX** — BESS, solar/genset, total.
- **Economia mensal** — peak avoided, variable cost (recharge or fuel), solar off-peak abatement, net.
- **Economia anual** — net.
- **Retorno financeiro** — O&M, annual net after O&M, simple payback, escalated payback.
- **Economia acumulada 12 anos** — sum of the 12 annual flows (with escalation/degradation).
- **VPL** and **TIR**.

### Worked example — current default inputs

Values rounded for display (workbook keeps full precision internally; the displayed `R$ 446k`, `R$ 1.07M`, etc. correspond to `446,429`, `1,066,515`, etc.).

| | C1 BESS | C2 Solar→BESS | C3 Solar→FP | C4 Diesel |
|---|---:|---:|---:|---:|
| Capacidade BESS (kWh) | 248 | 248 | 248 | — |
| Potência inversor/gerador (kW) | 82.7 | 82.7 | 82.7 | 71.4 |
| kVA nominal | — | — | — | 107.1 |
| kWp solar | — | 42 | 292 | — |
| CAPEX total | R$ 446k | R$ 530k | R$ 1.030M | R$ 220k |
| Economia mensal | R$ 14,546 | R$ 17,570 | R$ 35,505 | R$ 8,307 |
| Economia anual bruta | R$ 174,549 | R$ 210,837 | R$ 426,060 | R$ 99,684 |
| O&M anual | R$ 2,232 | R$ 2,652 | R$ 5,152 | R$ 10,000 |
| Economia anual líquida | R$ 172,316 | R$ 208,184 | R$ 420,908 | R$ 89,684 |
| Payback simples | 2a 7m | 2a 7m | 2a 5m | 2a 5m |
| Payback com reajuste | 2a 6m | 2a 5m | 2a 4m | 2a 4m |
| Cumulativo 12 anos | R$ 2.73M | R$ 3.30M | R$ 6.67M | R$ 1.66M |
| **VPL @ 12%** | R$ 875k | R$ 1.07M | **R$ 2.20M** | R$ 559k |
| **TIR** | 42.5% | 43.2% | 44.9% | **47.5%** |

**Verdict the calculator delivers** for the default profile:
- **C3 wins on NPV and cumulative savings** (the big solar array nearly eliminates the off-peak bill on top of the load shift).
- **C4 has the highest IRR** thanks to low CAPEX — attractive when capital is constrained but absolute returns are the smallest.
- **C1 is the lowest-friction BESS entry point** — works with no rooftop space or solar permits.

---

## 7. Modelling caveats (read before implementing)

These are simplifications and edge cases an implementer needs to make explicit in code.

### Modelling resolution & energy accounting

1. **Implicit off-peak window.** The model treats peak as `C12 · C13` hours (default 63 h/mês) and assumes **everything else** is off-peak. No daily timestep — solar generation is monthly, BESS cycles are monthly. Solar self-consumption vs export is not modelled at sub-monthly resolution.

2. **Solar energy treated as monthly-fungible.** The cascade `BESS recharge → off-peak abatement` assumes solar generation can be redirected to either bucket regardless of *when* it happens during the day. In reality solar peaks at midday, BESS typically charges off-peak (overnight), and off-peak consumption distributes around the clock — alignment requires either (a) direct daytime self-consumption, or (b) net-metering / GD compensation credits.

3. **Solar surplus is silently discarded.** In C2/C3 the cascade is BESS recharge → off-peak abatement → **nothing**. Any monthly generation beyond off-peak consumption is wasted. If GD compensation rules (Lei 14.300/2022) credit the surplus, the model understates C2/C3 returns.

4. **No seasonal variation.** `geração_por_kWp` (`C25`, default 120 kWh/kWp/mês) is a single yearly average. Real plants vary ~±20% across the year (worst in winter for southern Brazil). A worst-month sizing rule would call for ~25–35% more kWp than this model suggests.

### Tariffs & demand charges

5. **Demand charges (TUSD demanda contratada) not modelled.** Grupo A customers pay a separate kW-demand fee on top of energy. Peak-shaving with a BESS can also reduce the contracted demand, producing **additional savings not captured here**. This is a material gap for commercial proposals.

6. **No tariff modality distinction.** Grupo A has Verde (single demand tariff, no peak/off-peak split on demand) and Azul (separate demand tariffs for peak and off-peak). The model treats both as if peak/off-peak **energy** tariffs are the only differentiator, ignoring the demand-charge differences between modalities.

### BESS sizing & degradation

7. **Suggested vs installed kWp.** `C29`/`C34` are pure calculations; `C30`/`C35` are **manual rounded-up overrides** that drive both CAPEX and monthly generation. A backend must expose `kWp_installed` as a separate input, defaulting to `CEIL(kWp_suggested)` or similar.

8. **Battery degradation reduces savings, not capacity.** The 2%/ano factor multiplies annual savings, but BESS nameplate and inverter power stay constant — no resizing, no replacement. After 12 years residual capacity is `(1−0.02)^11 ≈ 80%` of nominal (compounded — *not* linear 1−12·0.02 = 76%) but this never feeds back into CAPEX or sizing.

9. **No end-of-life replacement or residual value.** Typical inverter life is 10–15 years and BESS warranties often cap at 10y; neither is modelled as a year-10/12 cash outflow, and no residual asset value is credited at year 12. Both omissions push results in opposite directions but neither is captured.

10. **O&M is flat percentage of CAPEX.** No escalation, no step-up for end-of-life. Reasonable for a 12-year horizon but conservative.

### Diesel-specific

11. **Diesel has its own escalator.** `C51` (default 7%) is independent of the tariff escalator `C42`. Defaults make them equal, but the model is built to let diesel inflate faster — a realistic Brazilian scenario.

12. **No diesel engine wear.** Consumption stays at `0.28 L/kWh` for all 12 years. Real gensets drift up several percent over their life, and `manutenção_anual` is also held flat.

### Financial assumptions

13. **Single discount rate (`TMA`).** No risk-adjusted rate per scenario; diesel's fuel-price risk and BESS's technology risk are not captured in differentiated discounts. If the customer's WACC differs from 12% the ranking can flip.

14. **Year-1 escalation exponent is 0.** All compounding uses `^(n−1)`, so year 1 is the un-escalated base year. A backend using `^n` will overstate NPV and IRR materially.

15. **"Payback com reajuste" is not discounted.** It scans the nominal escalated/degraded cumulative — not the discounted one. A true discounted payback would be longer.

16. **`MATCH(0, ·, 1)` requires monotonic ascending cumulative.** If any year's cumulative dips (e.g., a hypothetical year where O&M plus degradation outpaces escalation), the payback formula reports the wrong crossover. With current defaults this never happens, but a backend should validate monotonicity before trusting the trick.

17. **No taxes.** No ICMS, PIS/COFINS, IRPJ/CSLL, depreciation benefits, or REIDI considered. The cash flow is **pre-tax operational** only.

18. **No financing.** All scenarios assume 100% equity at t=0. No interest, no principal, no balloon — implementers wanting to compare cash-financed vs leased BESS need a separate model on top.

### Implementation pitfalls

19. **`Comparativo` payback rows pull strings.** The simple/escalated payback cells are strings like `"2 anos e 5 meses"` — not numbers. Don't try to compare or sort on them; recompute the underlying ratio if you need numeric ordering.

20. **`IFERROR` everywhere.** The workbook is liberal with `IFERROR(…, 0)` to keep cells displayable when an input is missing. A backend implementation should fail loud instead — silent zeros hide misconfiguration.

21. **NPV's t=0 convention.** Excel's `NPV(rate, cf1..cfn)` treats `cf1` as occurring at end of period 1. The workbook adds the year-0 outflow *outside* the call: `NPV(rate, cf1..cf12) + cf0`. A backend using a library where `npv(rate, cfs)` treats `cfs[0]` as `t=0` (e.g. some Python/JS implementations) must not also add `cf0`.
