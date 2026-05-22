# Official Results Registry

## ACSIncome ethical audit — CA + MS subsample

**Source of truth**: `notebooks/progetto_etica_dati.ipynb` (executed end-to-end, all outputs saved). Questo file rispecchia i risultati del notebook per riferimento rapido.

## Setup

- Dataset: ACSIncome (Folktables, Ding et al. 2021), source ACS 2018 1-year person-level
- Stati: California + Mississippi
- Dataset completo per stato: CA 195,665 / MS 13,189
- Subsample bilanciato: 26,378 record (13,189 per stato, su dimensione MS)
- Distribuzione target nel subsample: 34% True (`PINCP > 50k`) / 66% False
- Distribuzione SEX nel subsample: M 51.8% / F 48.2%
- Train/test split: 70/30, stratificato su target, seed 42
- Modelli: `LogisticRegression` + `GradientBoostingClassifier` (sklearn defaults)
- Attributo sensibile primario: SEX (1=M, 2=F); secondari: RAC1P, STATE
- Feature input modello: 10 (STATE escluso dalle feature)

## Sezione 3 — EDA: tassi positivi e gap

### Tasso `income > 50k` per gruppo

| Gruppo | P(income>50k) |
|---|---|
| SEX=1 (M) | 0.411 |
| SEX=2 (F) | 0.264 |
| RAC1P=1 (White) | 0.381 |
| RAC1P=2 (Black) | 0.179 |
| RAC1P=3 | 0.231 |
| RAC1P=5 | 0.167 |
| RAC1P=6 (Asian) | 0.478 |
| RAC1P=7 | 0.140 |
| RAC1P=8 | 0.190 |
| RAC1P=9 | 0.334 |
| STATE=CA | 0.411 |
| STATE=MS | 0.270 |

### Gap aggregati

- Gender gap: M 41.1% − F 26.4% = **14.7 pp**
- Range razziale: RAC1P=6 47.8% − RAC1P=7 14.0% = **33.8 pp** (~34 pp)
- State gap: CA 41.1% − MS 27.0% = **14.1 pp**

### Intersezionali SEX × STATE

| | CA | MS |
|---|---|---|
| M | 0.462 | 0.357 |
| F | 0.353 | 0.180 |
| **Gap M−F** | **10.9 pp** | **17.7 pp** |

## Sezione 3 — Test statistici

| Test | Variabili | Statistica | dof | p-value |
|---|---|---|---|---|
| Chi² | SEX × target | 629.03 | 1 | 8.12e-139 |
| Chi² | RAC1P × target | 1046.75 | 7 | 9.53e-222 |
| Chi² | STATE × target | 586.78 | 1 | 1.26e-129 |
| KS | AGEP (target=T vs F) | D=0.260 | — | < 0.001 |
| t-test | AGEP (M vs F) | t=1.05 | — | 0.293 |

AGEP media: M = 43.26, F = 43.07 (differenza non significativa). Tutti i Chi² e KS: p < α=0.05.

## Sezione 4 — Modelli baseline (test set globale)

| Metrica | Logistic Regression | Gradient Boosting |
|---|---|---|
| accuracy | 0.7766 | 0.8122 |
| precision | 0.7138 | 0.7593 |
| recall | 0.5733 | 0.6561 |
| f1 | 0.6359 | 0.7040 |
| roc_auc | 0.8432 | 0.8820 |

## Sezione 5 — Performance disaggregata

### Per SEX

| Modello | Metrica | M | F | gap M−F |
|---|---|---|---|---|
| LR | accuracy | 0.764 | 0.789 | −0.025 |
| LR | precision | 0.721 | 0.694 | 0.027 |
| LR | recall | 0.686 | 0.394 | **0.292** |
| LR | f1 | 0.703 | 0.503 | 0.200 |
| GB | accuracy | 0.794 | 0.832 | −0.038 |
| GB | precision | 0.765 | 0.748 | 0.018 |
| GB | recall | 0.711 | 0.569 | **0.142** |
| GB | f1 | 0.737 | 0.646 | 0.091 |

### Per RAC1P (Gradient Boosting)

| RAC1P | n_test | accuracy | precision | recall |
|---|---|---|---|---|
| 5 | 7 | 1.000 | 1.000 | 1.000 |
| 7 | 15 | 1.000 | 1.000 | 1.000 |
| 9 | 189 | 0.862 | 0.737 | 0.636 |
| 2 | 1368 | 0.858 | 0.779 | 0.401 |
| 3 | 45 | 0.844 | 0.714 | 0.500 |
| 8 | 506 | 0.844 | 0.733 | 0.330 |
| 1 | 5044 | 0.799 | 0.750 | 0.696 |
| 6 | 740 | 0.774 | 0.806 | 0.731 |

### Per STATE

| Modello | STATE | n_test | accuracy | precision | recall | f1 |
|---|---|---|---|---|---|---|
| LR | CA | 3957 | 0.770 | 0.806 | 0.579 | 0.674 |
| LR | MS | 3957 | 0.783 | 0.605 | 0.565 | 0.585 |
| GB | CA | 3957 | 0.820 | 0.823 | 0.716 | 0.766 |
| GB | MS | 3957 | 0.804 | 0.660 | 0.564 | 0.608 |

## Sezione 6 — Metriche di equità

### Gradient Boosting × SEX (1=privilegiato, 2=non-privilegiato)

| # | Metrica | M (SEX=1) | F (SEX=2) | Δ / valore |
|---|---|---|---|---|
| 1 | Demographic Parity P(Ŷ=1) | 0.378 | 0.206 | DP_diff = 0.172 |
| 2 | Equal Opportunity (TPR) | 0.711 | 0.569 | EOpp_diff = 0.142 |
| 3 | Equalized Odds (TPR, FPR) | (0.711, 0.150) | (0.569, 0.071) | max_diff = 0.142 |
| 4 | Disparate Impact P(F)/P(M) | — | — | **DI = 0.544** (regola 80% VIOLATA) |
| 5 | Predictive Parity (PPV) | 0.765 | 0.748 | PredP_diff = 0.018 |
| 6 | Calibration gap medio | 0.022 | 0.041 | — |

### Logistic Regression × SEX

| # | Metrica | M | F | Δ / valore |
|---|---|---|---|---|
| 1 | Demographic Parity P(Ŷ=1) | 0.387 | 0.154 | DP_diff = 0.233 |
| 2 | Equal Opportunity (TPR) | 0.686 | 0.394 | EOpp_diff = 0.292 |
| 3 | Equalized Odds (TPR, FPR) | (0.686, 0.182) | (0.394, 0.064) | max_diff = 0.292 |
| 4 | Disparate Impact P(F)/P(M) | — | — | **DI = 0.397** (regola 80% VIOLATA) |
| 5 | Predictive Parity (PPV) | 0.721 | 0.694 | PredP_diff = 0.027 |
| 6 | Calibration gap medio | 0.018 | 0.057 | — |

### Gradient Boosting × STATE (CA=privilegiato, MS=non-privilegiato)

| # | Metrica | CA | MS | Δ / valore |
|---|---|---|---|---|
| 1 | Demographic Parity P(Ŷ=1) | 0.358 | 0.230 | DP_diff = 0.127 |
| 2 | Equal Opportunity (TPR) | 0.716 | 0.564 | EOpp_diff = 0.152 |
| 3 | Equalized Odds (TPR, FPR) | (0.716, 0.107) | (0.564, 0.107) | max_diff = 0.152 |
| 4 | Disparate Impact P(MS)/P(CA) | — | — | **DI = 0.645** (regola 80% VIOLATA) |
| 5 | Predictive Parity (PPV) | 0.823 | 0.660 | PredP_diff = 0.163 |
| 6 | Calibration gap medio | 0.053 | 0.024 | — |

### Riepilogo LR vs GB

| Metric | Attribute | LR | GB |
|---|---|---|---|
| DP_diff | SEX | 0.233 | 0.172 |
| EOpp_diff | SEX | 0.292 | 0.142 |
| EOdds_maxdiff | SEX | 0.292 | 0.142 |
| DisparateImpact | SEX | 0.397 | 0.544 |
| PredP_diff | SEX | 0.027 | 0.018 |
| DP_diff | STATE | 0.043 | 0.127 |
| EOpp_diff | STATE | 0.014 | 0.152 |
| EOdds_maxdiff | STATE | 0.039 | 0.152 |
| DisparateImpact | STATE | 0.853 | 0.645 |
| PredP_diff | STATE | 0.201 | 0.163 |

## Sezione 7 — SHAP top-5 (Gradient Boosting)

Top 5 feature per |SHAP value| medio:

1. WKHP
2. SCHL
3. AGEP
4. OCCP
5. SEX

Per SEX: punti femminili con SHAP value sistematicamente negativo, punti maschili positivo.

## Sezione 8 — Mitigation (Gradient Boosting × SEX)

| Metrica | baseline | Reweighing | ExpGrad | ThresholdOpt |
|---|---|---|---|---|
| accuracy | 0.812 | 0.805 | 0.811 | 0.797 |
| DP_diff | 0.172 | **0.056** | 0.126 | 0.093 |
| EOpp_diff | 0.142 | 0.050 | 0.069 | 0.052 |
| EOdds_maxdiff | 0.142 | 0.050 | 0.069 | 0.052 |
| DI | 0.544 | **0.827** | 0.649 | 0.741 |
| PredP_diff | 0.018 | 0.106 | 0.062 | 0.133 |

Reweighing porta DI dentro la regola 80% (0.827 ∈ [0.8, 1.25]) con costo accuracy −0.7 pp.

## Sezione 10 — k-anonymity pre-generalizzazione

QID = {AGEP, SCHL, OCCP}

- Record totali: 26,378
- Tuple QID uniche: 18,826
- Record con k=1 (UNICI): 14,710 (**55.8%**)
- Record con k<5: 23,516 (89.2%)
- Record con k≥5: 2,862 (10.8%)
- Percentili k: p25=1, p50=1, p75=2, p90=5, p95=7

## Sezione 11 — k-anonymity post-generalizzazione + l-diversity

Generalizzazione: AGEP→6 fasce di 10y, SCHL→4 livelli edu, OCCP→7 macro-categorie (6 esplicite + catch-all `other`).

| Metrica | Originale | Post-generalizzazione |
|---|---|---|
| Tuple QID uniche | 18,826 | **164** |
| Record k=1 | 14,710 | **7** |
| Record k<5 | 23,516 | **21** |
| Record k≥5 | 2,862 | **26,357** |

Percentili k post-gen: p25=174, p50=274, p75=450, p90=842, p95=996.

### Soppressione finale (5-anonymity)

- Record soppressi (k<5): 21 (0.1%)
- Dataset 5-anonimo finale: 26,357 record (99.9%)
- min(k) finale = 6

### l-diversity (attributo sensibile PINCP)

- Classi di equivalenza totali: 152
- Classi con l=1 (omogenee, attaccabili): 6 (**3.9%**)
- Classi con l≥2 (sicure): 146 (96.1%)
- Record in classi l=1: **578** (**2.2%**)

## Sezione 12 — Differential Privacy

Query: età media (AGEP) per STATE.

- AGEP media reale CA (n=13,189): **42.7801**
- AGEP media reale MS (n=13,189): **43.5608**
- Sensibilità globale L1: Δf = 83 / 13,189 ≈ **0.006293**

### Output Laplace al variare di ε (1 run)

| ε | AGEP_CA reale | AGEP_CA DP | AGEP_MS reale | AGEP_MS DP |
|---|---|---|---|---|
| 0.01 | 42.7801 | 40.7580 | 43.5608 | 43.5426 |
| 0.10 | 42.7801 | 42.7954 | 43.5608 | 43.6088 |
| 1.00 | 42.7801 | 42.7842 | 43.5608 | 43.5609 |
| 10.00 | 42.7801 | 42.7801 | 43.5608 | 43.5610 |

L'errore scala come Δf/ε. A ε=1, errore medio query ≈ 0.006239 anni (Monte Carlo, 1000 simulazioni).

## Sezione 13 — GB addestrato sul dataset 5-anonimo

- Shape: `X_train_anon` (18449, 24), `X_test_anon` (7908, 24)
- Feature dopo one-hot encoding: 24

| Dataset | accuracy | precision | recall | F1 |
|---|---|---|---|---|
| Originale (Sez. 4) | 0.8122 | 0.7593 | 0.6561 | 0.7040 |
| 5-anonimo (Sez. 11) | **0.8064** | **0.7467** | 0.6520 | 0.6962 |

Costo k-anonymity sul modello GB: accuracy −0.58 pp, F1 −0.78 pp.
