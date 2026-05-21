# Dimensionamento Sistema Híbrido — Concepts & Business Logic

Plain-text capture of the business logic encoded in `docs/blueprints/sheets/dimensionamento_sistema_hibrido.xlsx`. This document is the source of truth for any future re-implementation (API, frontend calculator, spec, or audit). Cell references like `Premissas!B17` are kept inline so a reader can trace back to the workbook.

> **Companion model.** This calculator is the residential / small-commercial counterpart of `calculadora_BESS.xlsx` (Grupo A industrial). The two share design DNA — load-shift through batteries, optional PV — but differ fundamentally in *inputs* (appliance-level loads vs. monthly kWh blocks) and *regulatory model* (Grupo B Tarifa Branca + Lei 14.300 net metering vs. Grupo A peak/off-peak tariffs).

---

# ⚠ BUGS AND INCONSISTENCIES — TO REPORT TO THE SHEET AUTHOR

> This audit found **10 issues** in the workbook ranging from **inputs configured but never used** to **two sheets reporting different values for the same metric**. A backend implementation must decide case-by-case whether to faithfully reproduce or correct.
>
> **For the author:** every issue below cites exact cells, current behavior, and expected behavior. Severity legend:
> 🔴 **BUG** — formula produces a wrong number or ignores configured inputs
> ⚠ **INCONSISTENCY** — two cells claim to compute the same metric but disagree
> 🟡 **SMELL** — misleading labels, footguns, dead code, or silent failures (no wrong number today, but a hazard)

## Summary table

| # | Severity | Issue | Cells |
|---|---|---|---|
| **B1** | 🔴 BUG | Battery degradation never applied to cash flow | `Financeiro!B27` |
| **B2** | 🔴 BUG | Battery replacement never deducted at year 13 | `Financeiro!B28`, `B29` |
| **B3** | ⚠ INCONSISTENCY | C3 "Economia Ano 1" differs between sheets (R$ 7,382 vs R$ 8,174) | `C3!B36` vs `Comparativo!C26` |
| **B4** | 🔴 BUG | C3 hardcodes 30 days/month despite `B18=22` input | `C3!B22` (literal `30`); `B18` stranded |
| **B5** | ⚠ INCONSISTENCY | Static `tarifa_compensável` (0.9) coexists with Comparativo ramp (0.6→0.75→0.9) | `Premissas!B80/B81` vs `Comparativo!C15` |
| **B6** | ⚠ POSSIBLY OUTDATED | Lei 14.300 ramp caps at 0.9 instead of reaching 1.0 (2029+) | `Comparativo!C15` IF chain |
| **B7** | 🟡 SMELL | C3 informational decomposition does **not** sum to the net (16.98 − 11.19 − 3.06 = 2.73 ≠ 4.70) | `C3!B24:B27` |
| **B8** | 🟡 SMELL | Horizon stored as **text string** `"24"` — `VALUE()` coercion everywhere | `Financeiro!B23` |
| **B9** | 🟡 SMELL | Validation flags (`⚠ INCOMPATÍVEL`) not propagated to cash flow | `Premissas!B53`, `B57` |
| **B10** | 🟡 SMELL | `Fluxo Descontado Acumulado` rows computed but unused | `Comparativo` rows 10/11/19/20 |

## Expanded details

### 🔴 B1 — Battery degradation is never applied

- **Cell:** `Financeiro!B27` (input, default 2%/ano)
- **What happens today:** No formula in `Comparativo` references `B27`. The C2 economy row is `B31 · (1 + reajuste_tarifário)^n` and the C3 economy row (`C15` and right) similarly applies only tariff escalation. The `(1 − degradação)^n` factor is missing.
- **Expected:** Both yearly-economy formulas should multiply by `(1 − Financeiro!B27)^n` (or `^(n−1)`, depending on convention — but currently it's `^0` everywhere).
- **Impact:** With horizon = 24 and 2%/ano compounded, year-24 economy is overstated by `1 / (1−0.02)^23 ≈ 1.58×`. **NPV and IRR are materially inflated** for both C2 and C3.

### 🔴 B2 — Battery replacement is never deducted

- **Cells:** `Financeiro!B28` (R$ 12,000), `B29` (year 13)
- **What happens today:** No formula in `Comparativo` references `B28` or `B29`. The O&M row only has `−CAPEX · O&M% · (1 + inflação)^n` — same expression at every year, including year 13.
- **Expected:** Year `B29` should include an additional `−B28 · (1 + inflação)^B29` cash flow on the O&M (or a separate replacement) row.
- **Workbook comment** at `Financeiro!B23`: *"24 anos: inclui reposição de bateria"* — **the comment promises something the formulas don't deliver**.
- **Impact:** With horizon = 24, real cash flow should drop ~R$ 20,000 nominal (12k × 1.04¹³ ≈ 19,981) in year 13. NPV inflated by ~R$ 4,500 (discounted at 12% TMA).

### ⚠ B3 — C3 reports two different "Economia Ano 1" values

- **Cells:** `'C3 — Arbitragem + FV'!B36` (R$ 7,382/ano) vs. `Comparativo!C26` (R$ 8,174/ano)
- **What happens today:** `C3!B36 = B35·12` uses the static `tarifa_compensável_FP` from `Premissas!B81` (Fio B factor hardcoded to 0.9). `Comparativo!C26 = C15` recomputes the year-1 economy inline with the *ramped* Fio B factor that starts at 0.6.
- **Expected:** The two sheets should agree on year 1 economy. Either C3 sheet adopts the ramp, or Comparativo uses the static factor, or both reference a shared year-aware schedule.
- **Impact:** Comparativo's Payback Simples (4.4 anos) and the C3 sheet's Payback Simples (4.9 anos) disagree by ~10%. **Which number does a sales team show the customer?**

### 🔴 B4 — C3 hardcodes 30 days/month, ignores its own `B18=22` input

- **Cells:** `'C3 — Arbitragem + FV'!B18` (input "Dias Úteis no Mês" = 22) and `B22` (formula uses literal `30`)
- **What happens today:** `B22 = MIN(B10·30, B17)` — the literal `30` is baked into the formula. `B18 = 22` is set as an input but no formula on the C3 sheet references it. **Stranded input.**
- **Expected:** `B22 = MIN(B10 · B18, B17)`.
- **Impact:** A user editing `B18` (e.g., to 22) sees **no effect** on the result. Worse: C2 uses `B15 = 30` *and reads it*, so C2 and C3 use the same effective 30 days while *claiming* to use different conventions. If the intent was "C3 is more conservative with business days", the calculation doesn't reflect it — at 22 days, C3's headline would drop ~27%.

### ⚠ B5 — Static tarifa_compensável inconsistent with Comparativo ramp

- **Cells:** `Premissas!B80` (compensável ponta = 1.082) and `B81` (compensável FP = 0.122) — both use hardcoded `0.9` for the Fio B factor — vs. `Comparativo!C15` which uses a year-dependent ramp.
- **What happens today:** Two coexisting Lei 14.300 implementations: a static one in Premissas and a ramped one inline in Comparativo. The C3 sheet reads the static one (and so its outputs differ from Comparativo — see B3).
- **Expected:** Single, year-aware Fio B schedule parameterized by project start year, shared across all consumers.
- **Impact:** Drives B3 directly.

### ⚠ B6 — Lei 14.300 ramp caps at 0.9 instead of reaching 1.0

- **Cell:** `Comparativo!C15` (the IF chain `IF(year≤1, 0.6, IF(year≤2, 0.75, 0.9))`)
- **What happens today:** The Fio B factor ramps 0.6 → 0.75 → 0.9 and then stays at 0.9 forever (years 3..24).
- **Expected:** Actual **Lei 14.300/2022** schedule reaches **100% Fio B from 2029**. If the project starts in 2026 (which the 0.6 → 0.75 → 0.9 sequence implies — those map to 2026, 2027, 2028), year 4 onwards should be 1.0, not 0.9. The current ramp also missing the earlier values (2023=0.15, 2024=0.30, 2025=0.45) — fine if start year is 2026 or later, broken otherwise.
- **Impact:** Slight overstatement of C3 from year 4 (2029) onwards; complete model break if the project starts before 2026.

### 🟡 B7 — C3 informational decomposition does not equal the net

- **Cells:** `'C3 — Arbitragem + FV'!B24:B27`
- **What happens today:** Four cells *labeled as if they decompose*:
  - B24 "Economia Bruta — Compensação FV" = R$ 16.98 = injetada · tarifa_FP
  - B25 "Custo Fio B sobre Energia Compensada" = R$ 11.19 = injetada · Fio_B (face value, **no ICMS gross-up, no 0.9 factor**)
  - B26 "Custo ICMS sobre Energia Compensada" = R$ 3.06 = injetada · tarifa_FP · ICMS (**naive — applies ICMS to the full FP rate, not just the non-compensated portion**)
  - B27 "Economia Líquida — Compensação FV" = R$ 4.70 = injetada · tarifa_compensável_FP
- **Reader expectation:** `B27 = B24 − B25 − B26`. Reality: `16.98 − 11.19 − 3.06 = 2.73 ≠ 4.70`.
- **Root cause:** B25/B26 use *naïve* face-value formulas; B27 uses the proper Lei 14.300 net tariff. They aren't a tight decomposition.
- **Expected:** Either (a) fix B25/B26 to genuinely decompose B27 (`B25 = injetada · Fio_B/(1−ICMS) · 0.9`, `B26 = …` accordingly), or (b) re-label B24/B25/B26 as "approximate / informational breakdown — does not reconcile to net".
- **Impact:** Confuses anyone auditing the C3 sheet; the labels are a trap.

### 🟡 B8 — Horizon stored as a text string

- **Cell:** `Financeiro!B23` (value: literal `"24"`, type: string)
- **What happens today:** Every consumer wraps in `VALUE(Financeiro!$B$23)` to coerce to number. `OFFSET(B9, 0, Financeiro!B23)` relies on Excel's implicit text-to-number coercion.
- **Expected:** Numeric input with data validation (e.g., dropdown list of `{12, 24}`).
- **Impact:** Typing a non-numeric horizon (e.g., "doze") would silently break every cash-flow formula. A backend implementation that reads `B23` as a string and forgets to parse will crash on the comparison.

### 🟡 B9 — Validation flags not propagated

- **Cells:** `Premissas!B53` (voltage check, returns "✓ …" or "⚠ Tensão INCOMPATÍVEL") and `Premissas!B57` (current check, similar)
- **What happens today:** The two strings live only on `Premissas`. No downstream cell checks their prefix.
- **Expected:** Either propagate to `Comparativo` as visible indicator rows, or guard the cash-flow formulas with `IF(LEFT(Premissas!B53,1) = "⚠", …)`.
- **Impact:** A user can produce a beautiful NPV/IRR report with the underlying bank voltage or current incompatible — and never notice. Especially dangerous given the validations are the only quality check in the workbook.

### 🟡 B10 — Discounted cumulative cash flow is dead code

- **Cells:** `Comparativo` rows 10, 11 (C2 discounted + acumulado) and 19, 20 (C3 same)
- **What happens today:** Both scenarios compute `Fluxo Descontado = Fluxo Líquido / (1+TMA)^n` and its accumulator — but **no indicator surfaces them**. The "Economia Acumulada" indicator (`B30`, `C30`) uses the *nominal* accumulator (rows 9, 18).
- **Expected:** Either surface a "Discounted Cumulative at Horizon" indicator (would also enable a true discounted payback), or remove the unused rows.
- **Impact:** None today; opportunity to add a true discounted payback indicator.

---

## 1. Purpose & domain context

### What the calculator solves

Brazilian residential and small-commercial customers (Grupo B — low voltage) sizing a **hybrid inverter + battery bank + optional PV array** for one of three strategic outcomes:

| Code | Strategy | Customer value |
|---|---|---|
| **C1 — Backup Puro** | Battery + inverter sized to keep essential loads running during grid outages | Continuity of supply — value is *qualitative* (depends on how much the customer is willing to pay for uptime) |
| **C2 — Arbitragem Tarifária (Tarifa Branca)** | Battery charges off-peak from grid, discharges during the ~3-hour peak window | Spread between Tarifa Branca peak/off-peak rates (~3× ratio) |
| **C3 — Arbitragem + FV (Lei 14.300)** | Above + PV array: solar self-consumption + solar-charged battery + grid-injected surplus | Compounds: avoided peak draw, avoided off-peak draw, and partial compensation for injected surplus |

C1 has **no economy model** — it only sizes hardware. C2 and C3 are the financially modelled scenarios.

### Workbook layout

| Tab | Role |
|---|---|
| `Premissas` | Loads, BESS sizing, validations, tariffs, PV cascade (one giant page) |
| `Financeiro` | CAPEX modes + financial assumptions |
| `C1 — Backup` | Passive: pulls sizing from Premissas, computes real autonomy, no economy |
| `C2 — Arbitragem` | Tariff arbitrage on Tarifa Branca, no PV |
| `C3 — Arbitragem + FV` | Arbitrage + PV self-consumption + Lei 14.300 compensation |
| `Comparativo` | 25-column year-by-year cash flow (C2 & C3 only) → Payback, NPV, IRR, cumulative |

Color code (per `Premissas!A2`): **blue = user input · black = formula · green = cross-sheet reference · yellow = key flag/assumption**.

---

## 2. Inputs

### Bloco 1 — Inverter sizing (load inventory) — `Premissas!A4:G26`

Unlike the BESS calculator that takes one aggregated peak kWh figure, this model **starts from individual appliances**.

| Field | Range | Type | Default | Role |
|---|---|---|---|---|
| Equipment rows (up to 10) | `A6:G15` | mixed | 5 items populated | Each row: name · Pot. Unit (W) · Qtd · Tipo carga · Fator partida · → Pot. contínua / Pico calculated |
| η inversor | `B20` | % | 95% | Inverter efficiency |
| Margem segurança | `B21` | % | 20% | Safety headroom above continuous power |
| Faixa DC mín/máx | `B22/B23` | V | 40 / 60 | Inverter input voltage window (datasheet) |

**Per-equipment derivations:**
```
Pot. contínua (W) = Pot. Unit · Qtd                  (F6:F15)
Pico (W)          = Pot. Unit · Qtd · Fator partida  (G6:G15)
```

**Tipo de carga** field is descriptive only ("Resistiva", "Indutiva", "Eletrônica") — not used in any formula. The `Fator Partida` column carries the actual numeric multiplier (1 for resistive, 2 for inductive, etc.) and is what's used in the math.

**Aggregates:**
```
Pot. Contínua Total (W) = SUM(F6:F15)                (B17, default 2,700 W)
Pico de Partida Total   = SUM(G6:G15)                (B18, default 3,300 W)
```

### Bloco 2 — Battery bank — `Premissas!A28:D57`

| Field | Cell | Default | Role |
|---|---|---|---|
| Tensão nominal do banco | `B30` | 48 V | Bank DC voltage — must lie inside the inverter DC window |
| Tensão nominal unitária | `B31` | 48 V | Per-battery DC voltage |
| Capacidade unitária | `B32` | 100 Ah | Per-battery Ah |
| Corrente máxima de descarga unit. | `B34` | 100 A | Per-battery limit |
| DoD | `B37` | 90% | Profundidade de descarga (LFP-typical) |
| η round-trip | `B38` | 91% | Single lumped factor: ohmic + electrochemical + BMS + thermal losses |
| Autonomia desejada | `B39` | 4 h | Sized against essential loads |

Battery unit kWh is computed: `B33 = B31·B32/1000 = 4.8 kWh`.

### Bloco 4 — Tariffs (Grupo B) — `Premissas!A60:D81`

The numbering quirk in the workbook: **Bloco 3** is PV (`A83`), but it appears *after* Bloco 4 (Tariffs, `A60`) visually because the PV block depends on Bloco 4's flag for its sizing rule.

| Field | Cell | Default | Role |
|---|---|---|---|
| **Modalidade Tarifária** | `B61` | "Branca" | **Switch flag:** "Convencional" or "Branca" — drives multiple downstream IFs |
| Tarifa convencional | `B64` | R$ 0.950/kWh | Used only when modalidade = "Convencional" |
| Consumo mensal — Convencional | `B65` | 400 kWh | "" |
| Tarifa ponta | `B68` | R$ 1.400/kWh | Used when modalidade = "Branca" |
| Tarifa fora ponta | `B69` | R$ 0.440/kWh | "" |
| Horas ponta/dia | `B70` | 3 h | Tarifa Branca peak window |
| Consumo ponta mensal | `B71` | 500 kWh | "" |
| Consumo fora ponta mensal | `B72` | 300 kWh | "" |
| ICMS | `B75` | 18% | State VAT rate |
| Fio B | `B76` | R$ 0.290/kWh | Wire-use fee — *not* compensated under Lei 14.300 |

### Bloco 3 — PV (optional) — `Premissas!A83:D96`

| Field | Cell | Default | Role |
|---|---|---|---|
| **FV ativo?** | `B84` | "Sim" | **Switch flag:** "Sim" or "Não" — disables PV block entirely when "Não" |
| Geração por kWp | `B85` | 120 kWh/kWp/mês | Reference: Brazil 100–140 |
| Potência instalada real | `B86` | 5.45 kWp | User-overridable installed kWp |
| Fator de simultaneidade | `B89` | 40% | % of PV generation consumed instantly (no grid passthrough → no Fio B/ICMS) |

### Financeiro — CAPEX & financial assumptions

| Field | Cell | Default | Role |
|---|---|---|---|
| **Modo CAPEX** | `B5` | "Simplificado" | **Switch flag:** "Simplificado" (R$/kWh × capacity) or "Detalhado" (itemized) |
| R$/kWh armazenamento | `B8` | R$ 2,500 | Simplificado mode |
| Preço inversor / baterias / instalação / outros | `B14–B17` | 8k / 12k / 3.5k / 1.5k | Detalhado mode line items |
| **CAPEX total** | `B20` | R$ 36,000 | `IF(B5="Simplificado", B11, B18)` — the one number every cash flow uses |
| **Horizonte de análise** | `B23` | **"24" (string!)** | 12 or 24 anos — stored as text, wrapped in `VALUE()` everywhere |
| Reajuste tarifário | `B24` | 10%/ano | Annual tariff escalation |
| TMA | `B25` | 12%/ano | Discount rate for NPV |
| O&M | `B26` | 0.5%/ano of CAPEX | Annual maintenance |
| Degradação anual da bateria | `B27` | 2%/ano | 🔴 **Configured but never applied — see Bug B1** |
| Custo reposição da bateria | `B28` | R$ 12,000 | 🔴 **Configured but never deducted — see Bug B2** |
| Ano da reposição | `B29` | 13 | 🔴 **Configured but never used — see Bug B2** |
| Inflação | `B30` | 4%/ano | Applied only to O&M growth |

---

## 3. Derived global values

### Inverter sizing — `Premissas!B25:B26`

```
kW_contínua_mín_inversor = Pot_contínua_total / η_inversor · (1 + margem) / 1000   (B25)
                         = 2,700 / 0.95 · 1.20 / 1000 = 3.41 kW

kW_pico_mín_inversor     = Pot_pico_total / η_inversor · (1 + margem) / 1000       (B26)
                         = 3,300 / 0.95 · 1.20 / 1000 = 4.17 kW
```

The inverter must satisfy *both* — the continuous figure for steady operation, the peak figure for motor inrush.

### Battery bank — `Premissas!B42:B50`

```
Pot. cargas essenciais (W)         = B17                                         (B42 = 2,700)
Energia necessária por ciclo (kWh) = B42/1000 · autonomia                        (B43 = 10.8)
Capacidade nominal (kWh)           = B43 / DoD / η_round_trip                    (B44 = 13.19)
Capacidade em Ah (no tensão banco) = B44·1000 / V_banco                          (B45 = 275)
Baterias em série                  = ROUND(V_banco / V_unitária, 0), min 1       (B46 = 1)
Baterias em paralelo               = MAX(1, CEILING(Ah_necessário / Ah_unit, 1)) (B47 = 3)
Total baterias                     = série × paralelo                            (B48 = 3)
Capacidade nominal real (kWh)      = paralelo · kWh_unit                         (B49 = 14.40)
Capacidade utilizável (kWh)        = nominal real · DoD · η_round_trip           (B50 = 11.79)
```

Note: the workbook's `η round-trip` (B38, 91%) is a single lumped factor — unlike the BESS calculator which splits into `η_carga · η_bateria · η_descarga`. The simpler form is appropriate at this scale.

### Validations — `Premissas!B53, B57` (⚠ silent — see Bug B9)

Two validation strings (✓ or ⚠) that surface design errors immediately *on the Premissas tab* — but are not propagated:

```
B53 = IF(AND(V_banco ≥ DC_min, V_banco ≤ DC_max), "✓ Tensão compatível", "⚠ Tensão INCOMPATÍVEL")
B57 = IF(I_banco ≥ I_pico_inversor, "✓ Banco fornece corrente suficiente", "⚠ Banco INSUFICIENTE — aumentar paralelo")

where:
I_banco                        = paralelo · I_max_unit         (B54 = 300 A)
I_nominal_inversor             = kW_contínua · 1000 / V_banco  (B55 = 71 A)
I_pico_inversor                = kW_pico · 1000 / V_banco      (B56 = 87 A)
```

These live only in `Premissas` — they are not propagated to scenario tabs or Comparativo. A backend implementation should fail loud on `⚠` rather than letting downstream cells continue with invalid sizing.

### Tarifa Compensável (Lei 14.300) — `Premissas!B80:B81`

Lei 14.300/2022 changed Brazilian net metering: solar exported to the grid no longer fully offsets consumption — the customer must pay the **Fio B** (wire-use fee) portion of the bill on compensated kWh. The calculator captures this with a "compensable tariff":

```
Tarifa_compensável_ponta = tarifa_ponta − (Fio_B / (1 − ICMS)) · 0.9              (B80)
                         = 1.40 − (0.29 / 0.82) · 0.9 ≈ R$ 1.082/kWh

Tarifa_compensável_FP    = tarifa_FP − (Fio_B / (1 − ICMS)) · 0.9                 (B81)
                         = 0.44 − 0.354 · 0.9 ≈ R$ 0.122/kWh
```

The `0.9` is the **Fio B share** the customer pays. ⚠ The workbook hardcodes it to 0.9 *here*, but `Comparativo!C15` applies a **year-by-year ramp** (0.6 → 0.75 → 0.9) for C3 — **see Bugs B5 and B6**.

The grossing-up `Fio_B / (1 − ICMS)` accounts for the fact that the published Fio B is net-of-ICMS but it gets re-grossed when added back to the bill.

### PV cascade — `Premissas!B88:B96`

When `B84 = "Sim"`, PV generation is allocated through a deterministic waterfall:

```
Geração total (kWh/mês)            = kWp_real · geração_por_kWp                (B88 = 654)
Autoconsumo direto (kWh/mês)       = geração · fator_simultaneidade            (B90 = 261.6)
                                     ↳ no Fio B, no ICMS — direct consumption
Restante após autoconsumo          = geração − autoconsumo                     (B91 = 392.4)
FV → bateria (kWh/mês)             = MIN(restante, capacidade_utilizável · 30) (B92 = 353.8)
                                     ↳ PV charges battery, capped at battery throughput
Rede → bateria (kWh/mês)           = MAX(0, capacidade · 30 − FV→bat)          (B93 = 0)
                                     ↳ grid tops up battery if PV insufficient
FV excedente → rede (kWh/mês)      = MAX(0, restante − FV→bat)                 (B94 = 38.6)
                                     ↳ injected: pays Fio B + ICMS under Lei 14.300
Recarga bruta da rede (kWh/mês)    = (rede → bat) / η_round_trip               (B95 = 0)
                                     ↳ AC grid kWh needed to cover the round-trip losses
Consumo FP ajustado (kWh/mês)      = consumo_FP_original + recarga_bruta_rede  (B96 = 300)
```

If `B84 = "Não"`, every cell B88–B96 returns 0.

**Potência ideal sugerida (`B87`)** — recommended kWp depends on modalidade:
```
B87 = IF(modalidade = "Convencional",
         consumo_total / geração_por_kWp,
         (consumo_FP + capacidade_utilizável · 30) / geração_por_kWp)
```

The user-overridable installed kWp (`B86`) is what feeds CAPEX and generation — never `B87`.

---

## 4. Scenarios

### C1 — Backup Puro (`C1 — Backup`)

Passive sheet that **only sizes hardware** — no economy model. It pulls four values from Premissas and computes one derived autonomy:

```
B5 = Premissas!B17       (Pot. cargas essenciais — 2,700 W)
B6 = Premissas!B50       (Capacidade utilizável — 11.79 kWh)
B7 = Premissas!B48       (Nº total de baterias — 3)
B8 = B6 / (B5 / 1000)    (Autonomia real — 4.37 h)
B11 = Financeiro!B20     (CAPEX total — R$ 36,000)
```

The sheet's own footnote states: *"O Cenário 1 é backup puro — não gera economia na conta de energia. O valor para o cliente é a continuidade de fornecimento durante quedas de rede. O payback é qualitativo (depende de quanto o cliente valoriza a continuidade)."*

C1 is **not represented in Comparativo** — no NPV/IRR/payback row exists for it.

### C2 — Arbitragem (sem FV) — `C2 — Arbitragem`

Tariff arbitrage on Tarifa Branca: battery discharges during the 3-hour peak window, recharges off-peak.

**Energy displaced:**
```
Energia/ciclo (kWh)          = capacidade_utilizável                          (B14 = 11.79)
Dias úteis/mês               = 30  (input — but C3 hardcodes 30; see Bug B4)  (B15)
Energia deslocada/mês (kWh)  = MIN(energia/ciclo · dias, consumo_ponta)       (B16 = 353.8)
                               ↳ capped at the customer's actual peak draw
Energia para recarga (kWh)   = energia_deslocada / η_round_trip               (B17 = 388.8)
                               ↳ AC grid input including round-trip loss
```

**Monthly cash:**
```
Economia bruta (ponta evitada)     = energia_deslocada · tarifa_ponta         (B20 = R$ 495.33)
Custo recarga (fora ponta)         = energia_recarga · tarifa_FP              (B21 = R$ 171.07)
Economia líquida arbitragem        = bruta − recarga                          (B22 = R$ 324.26/mês)
```

The sheet includes a note (`A24:A27`) reminding that **Fio B and ICMS don't need a separate adjustment** for arbitrage — when the customer avoids drawing from the peak meter, they avoid the *full* peak tariff (Fio B and ICMS included); when they recharge off-peak, they pay the *full* off-peak tariff. The difference is already captured.

**Annual & payback:**
```
Economia anual (Ano 1) = B22 · 12              (B31 = R$ 3,891)
Payback simples        = CAPEX / Economia_ano1 (B33 = 9.3 anos)
```

### C3 — Arbitragem + FV — `C3 — Arbitragem + FV`

The most complex scenario. Composes **four economy streams**:

#### Stream 1 — Autoconsumo direto FV
```
Economia_autoconsumo = Premissas!B90 · tarifa_FP                            (B21 = R$ 115.10)
                       ↳ PV consumed instantly avoids the FP rate (assumes daytime consumption is FP)
```

#### Stream 2 — FV → Bateria → Ponta (the headline number)
```
Energia bateria descarregada na ponta = MIN(capacidade · 30, consumo_ponta) (B22 = 353.8)
Economia_bateria_ponta                = B22 · tarifa_ponta                  (B23 = R$ 495.33)
```

⚠ Note the literal `30` in B22's formula — this **ignores the C3 sheet's own `B18 = 22` "Dias Úteis" input** (Bug B4).

This treats the battery as discharging during peak hours regardless of whether the energy came from PV or grid. Combined with the PV cascade in Premissas, FV charges the battery first; only the shortfall (B93) is recharged from the grid.

#### Stream 3 — FV → Rede (compensação, Lei 14.300)
```
Economia bruta compensação      = injetada · tarifa_FP                      (B24 = R$ 16.98, informational)
Custo Fio B sobre compensada    = injetada · Fio_B                          (B25 = R$ 11.19, informational)
Custo ICMS sobre compensada     = injetada · tarifa_FP · ICMS               (B26 = R$ 3.06, informational)
Economia líquida compensação    = injetada · tarifa_compensável_FP          (B27 = R$ 4.70) ← THIS IS WHAT GETS USED
```

B24, B25, B26 are *informational only* — B27 stands alone using the pre-computed `tarifa_compensável_FP` from Premissas. **Do not subtract B25 and B26 from B24 to "derive" B27** — they don't actually decompose (`16.98 − 11.19 − 3.06 = 2.73 ≠ 4.70`). **See Bug B7** for the misleading-labels issue.

#### Stream 4 — Arbitragem complementar (rede → bateria, when FV insufficient)
```
Energia rede → bateria  = Premissas!B93                                     (B28 = 0 with defaults)
Custo recarga rede      = Premissas!B95 · tarifa_FP                         (B29 = 0)
Economia complementar   = (rede → bat · tarifa_ponta) − custo_recarga       (B30 = 0)
```

Zero with default inputs because PV is sized large enough to fully charge the battery. Becomes positive if PV is undersized relative to battery throughput.

#### Total
```
ECONOMIA LÍQUIDA TOTAL = stream_1 + stream_2 + stream_3 + stream_4          (B32 = R$ 615.13/mês)
Economia anual Ano 1   = B32 · 12                                           (B36 = R$ 7,382)
Payback simples        = CAPEX / Economia_ano1                              (B38 = 4.9 anos)
```

> ⚠ **Inconsistency (Bug B3 / B5):** `B36 = R$ 7,382` uses the *static* `tarifa_compensável` (Fio B factor = 0.9). The Comparativo tab recomputes year-1 economy with a *ramped* Fio B (0.6, 0.75, 0.9 for years 1, 2, 3+) and reports **R$ 8,174** for the same "Economia Ano 1" indicator. **The Comparativo number is what drives Payback, NPV, IRR.**

---

## 5. Financial model — Comparativo cash flow

`Comparativo` builds a year-by-year table from year 0 (column B) to year 25 (column AA). Columns beyond `Financeiro!B23` (horizon) are blanked via `IF(year > horizon, "", ...)`. Only C2 and C3 are modelled.

### Layout (per scenario)

| Row | Meaning |
|---|---|
| `CAPEX Inicial` | Year 0: `−CAPEX_total`; all other years: 0 |
| `Economia Anual` | See per-scenario formula below |
| `O&M Anual` | `−CAPEX · O&M% · (1 + inflação)^n` |
| `Fluxo Líquido` | Economia + O&M |
| `Fluxo Acumulado` | Cumulative sum of Fluxo Líquido |
| `Fluxo Descontado` | Fluxo Líquido `/ (1 + TMA)^n` |
| `Fluxo Descontado Acumulado` | Cumulative sum of Fluxo Descontado |

### C2 yearly economy (`row 6`, column index `n`)
```
Economia_C2_n = 'C2 — Arbitragem'!B31 · (1 + reajuste_tarifário)^n
```

No degradation (🔴 Bug B1), no Lei 14.300 dynamics — pure escalation from the static year-1 value.

### C3 yearly economy (`row 15`, column index `n`) — the elaborate one
```
Economia_C3_n = ( B21 + B23 + B30                                                ← streams 1, 2, 4 (static)
                + B9 · (tarifa_FP − (Fio_B / (1 − ICMS)) · fio_b_pct(n)) )       ← stream 3, ramped
              · 12 · (1 + reajuste_tarifário)^n

where fio_b_pct(n) = 0.6 if n ≤ 1,
                    0.75 if n ≤ 2,
                    0.9  otherwise   ← ⚠ caps at 0.9 — see Bug B6
```

This **re-derives** the compensação líquida inline rather than pulling `C3!B27`, because the static `tarifa_compensável_FP` in Premissas doesn't capture the Lei 14.300 ramp. ⚠ The two routes disagree — see Bugs B3, B5.

No degradation factor 🔴 (Bug B1).

### O&M (`rows 7, 16`)
```
O&M_n = −CAPEX · O&M% · (1 + inflação)^n
```

Exponent is `n` (not `n−1`). Year 1 already has 1 year of inflation. **Different convention from the BESS calculator's `n−1`** — worth noting for any backend that consumes both models.

🔴 **There is no year-13 step-up for battery replacement** (Bug B2) — the O&M row has the same expression at every year.

### NPV (`B28, C28`)
```
NPV = NPV(TMA, Fluxo_Líquido[year1..yearN]) + Fluxo_Líquido[year0]
```

Standard Excel convention: `NPV()` treats its first cash flow as t=1; year 0 is added outside.

### IRR (`B29, C29`)
```
TIR = IRR(Fluxo_Líquido[year0..yearN], 10%)
```

Direct Excel IRR with `0.10` as the initial guess.

### Payback Simples (`B27, C27`)
```
Payback = CAPEX / Economia_ano1
```

Where `Economia_ano1` for C2 is `'C2 — Arbitragem'!B31` (= R$ 3,891), and for C3 is **the Comparativo C15** (= R$ 8,174 with the Lei 14.300 ramp), *not* `'C3 — Arbitragem + FV'!B36` (= R$ 7,382). Consequence: C3's payback shown in Comparativo (4.4 anos) differs from C3's payback shown in the C3 sheet (4.9 anos).

### Economia Acumulada (`B30, C30`)
```
Economia_acum = OFFSET(Fluxo_Acumulado[year0], 0, horizon)
```

Reads the cumulative **nominal** flow at the horizon column. Uses `OFFSET` so the formula adapts when the horizon input changes from "24" to "12".

🟡 **The `Fluxo Descontado Acumulado` row is computed but never surfaced as an indicator** (Bug B10) — there is no "Discounted Economia Acumulada" line, and no true discounted payback metric.

---

## 6. Outputs

### `Comparativo!A23:C31` — Resumo dos Indicadores Financeiros

| Indicator | C2 (default) | C3 (default) |
|---|---:|---:|
| CAPEX Total | R$ 36,000 | R$ 36,000 |
| Economia Líquida — Ano 1 | R$ 3,891 | **R$ 8,174** (ramped) |
| Payback Simples (anos) | 9.3 | 4.4 |
| VPL (R$) | R$ 37,190 | R$ 104,661 |
| TIR (% a.a.) | 20.1% | 31.9% |
| Economia Acumulada (24 anos) | R$ 335,472 | R$ 675,343 |
| Horizonte | 24 | 24 |

**Verdict the calculator delivers** for the default profile: C3 dominates on every metric — payback halves, NPV nearly triples, IRR jumps ~12 percentage points. The only context where C2 wins is when roof space, permits, or upfront PV cost rule out the array.

> ⚠ These numbers are **inflated** by Bugs B1 (no degradation) and B2 (no battery replacement). A corrected model would shave several percentage points off the IRR and slow the payback by ~6–12 months. Treat the indicators as upper bounds, not commitments.

### Per-scenario sheet outputs

| Sheet | Headline outputs |
|---|---|
| `C1 — Backup` | Autonomia real (h), CAPEX, *no economy* |
| `C2 — Arbitragem` | Economia mensal/anual, Payback simples |
| `C3 — Arbitragem + FV` | Economia mensal/anual (4 streams), Payback simples — **using static Lei 14.300 factor 0.9** |

---

## 7. Modelling caveats

> **Bugs and inconsistencies live at the top of this document** under "BUGS AND INCONSISTENCIES — TO REPORT TO THE SHEET AUTHOR" (issues B1–B10). This section covers only **deliberate simplifications** and **implementation pitfalls** that a backend re-implementation needs to know.

### Deliberate simplifications

1. **Monthly aggregation only.** No daily timestep — PV generation is monthly, battery cycles are monthly. Solar self-consumption vs. battery charging vs. grid injection at sub-monthly resolution is not modelled.

2. **Fator de simultaneidade is a single number.** Real self-consumption ratio depends on load profile, PV size, and season. The workbook uses one input (40%) for all 24 years.

3. **PV generation flat for 24 years.** No PV panel degradation (~0.5%/year typical). PV output is `kWp · 120 kWh/kWp/mês` in year 1 and year 24 alike.

4. **No seasonal variation.** `Geração por kWp` (`B85`, 120 kWh/kWp/mês) is a yearly average. Real installations vary ~±20% across seasons.

5. **C1 (backup) has no quantitative financial output.** It appears in `Premissas` / `Financeiro` / its own tab but not in `Comparativo`. By design — value of backup is qualitative — but means a hybrid system that's *both* backup-grade and arbitrage-economic can't be evaluated as one entity in this workbook.

6. **Autoconsumo direto FV priced at off-peak tariff.** `C3!B21 = autoconsumo · tarifa_FP` assumes the daytime solar generation displaces only *off-peak* consumption (which it does, since Tarifa Branca peak is 18–21h while solar peaks at midday). Correct but worth flagging — does not capture cases where peak hours overlap with sun (e.g., afternoon peak in some concessionárias).

7. **No taxes.** No IRPF, depreciation benefits, REIDI, or municipal incentives modelled. The cash flow is **pre-tax operational**.

8. **No financing.** All scenarios assume 100% equity at t=0.

9. **Modalidade flag is binary** (`Convencional`/`Branca`). Other Grupo B modalities exist (e.g., baixa renda) but are not handled.

10. **CAPEX mode flag (`Financeiro!B5`)** — switching between "Simplificado" and "Detalhado" can produce very different CAPEX (R$ 36k simplified vs R$ 25k detailed with defaults). A backend should require the user to explicitly choose one rather than defaulting silently.

11. **Tipo de carga is descriptive only.** The text field ("Resistiva", "Indutiva", etc.) is not used in any formula — only `Fator Partida` drives the math. A backend can drop the type field or use it only as UI guidance.

12. **Inflação applied only to O&M.** `Financeiro!B30` (4%/ano) compounds only the O&M line — not maintenance, replacement, or any other operational cost. Reasonable since most other costs are static, but worth noting.

### Implementation pitfalls

13. **Excel `NPV()` t=1 convention.** `NPV(rate, cf1..cfn)` treats `cf1` as occurring at end of period 1. The workbook adds `cf0` outside the call (`NPV(…) + B5` or `NPV(…) + B14`). A backend using a library where `npv(rate, cfs)` treats `cfs[0]` as t=0 (some Python/JS implementations) **must not also add `cf0`** or it gets counted twice.

14. **`IFERROR(…, 0)` everywhere.** The workbook swallows formula errors as zero — a backend should fail loud instead so misconfiguration doesn't hide.

15. **`OFFSET` for Economia Acumulada** assumes `Financeiro!B23` is the same column index as the year. With column B = year 0, the formula `OFFSET(B9, 0, B23)` lands on the cumulative for year `B23`. Works because horizons are integers and column layout is dense; a backend should explicitly index by year, not by column offset.

16. **Modalidade & FV flags use string comparison.** `"Convencional"`, `"Branca"`, `"Sim"`, `"Não"` are compared as strings — type these as enums, not free text.

17. **Comparativo column-to-year mapping.** Column B = year 0, C = year 1, …, AA = year 25 (26 columns supports horizons up to 25). `COLUMN() - 2` is used as the year index. A backend should expose years as numeric, not column offsets.

18. **Two different escalation conventions across the workbook family.** This workbook uses exponent `^n` (year 1 has 1 year of escalation). The companion `calculadora_BESS.xlsx` uses `^(n−1)` (year 1 has zero years of escalation). A backend that consumes both must keep the convention straight per model.
