# SYLLABUS MAPPING — Mapping tecniche del progetto ↔ slide del prof Rosati

Questo documento è la "fonte di verità" per quali tecniche sono
ammissibili nel notebook. Ogni tecnica deve essere riconducibile a una
slide specifica. Se una tecnica non è in questa lista, **non va usata
senza prima averla approvata in chat con Claude**.

## File slide di riferimento (corso Rosati, a.a. 2025/2026)

1. `analisi-dati.pdf` (102 slide) — metodi quantitativi di base
2. `bias.pdf` (51 slide) — modulo Bias ed equità
3. `privacy.pdf` (116 slide) — modulo Protezione della privacy

## Tecniche AMMISSIBILI — modulo Analisi Dati

| Tecnica | Slide | Uso nel progetto |
|---|---|---|
| Statistica descrittiva (media, mediana, varianza, range) | analisi 10-19 | EDA |
| Istogrammi, grafici a barre, boxplot | analisi 20-24 | Plot EDA |
| Chi-quadro (variabili categoriche) | bias 21 | Test indipendenza sensibile/target |
| Kolmogorov-Smirnov (variabili continue) | bias 21 | Confronto distribuzioni tra gruppi |
| t-test / ANOVA | bias 21 | Confronto medie tra gruppi |
| Regressione logistica | analisi 60-69 | Modello baseline interpretabile |
| Matrice di confusione | analisi 69 | Valutazione modello |
| K-means | analisi 75-89 | Eventuale clustering esplorativo (opzionale) |
| PCA | analisi 90-100 | Riduzione dimensionalità (opzionale) |
| Encoding categoriche | analisi 101-102 | Preprocessing obbligatorio |

## Tecniche AMMISSIBILI — modulo Bias ed equità

### Tipi di bias (per discussione/identificazione, slide 5-15)
- Data-to-algorithm: measurement, omitted variable, representation,
  aggregation (Simpson's paradox), sampling, longitudinal, linking
- Algorithm-to-user: algorithmic, ranking, popularity, emergent,
  evaluation
- User-to-data: historical, population, self-selection, social,
  behavioral, temporal, content production

→ Nel progetto: identificare almeno **historical bias** (slide 14) e
  **representation bias** (slide 8) nel dataset ACSIncome, possibilmente
  anche **measurement bias** (slide 6).

### Classificazione discriminazioni (slide 16-17)
- Explainable vs Unexplainable (Direct/Indirect)
- Systemic discrimination
- Statistical discrimination

### Misure di bias (slide 18-21)
- Analisi distribuzione (frequenze, condizionata, visualizzazioni
  comparative, densità) — **OBBLIGATORIO** in EDA
- Test statistici Chi-quadro / KS / t-test — **CONSIGLIATO**

### Metriche di equità (slide 22-28) — TUTTE E 6 IMPLEMENTATE
| Metrica | Slide | Formula |
|---|---|---|
| Demographic parity | 23 | P(Ŷ=1\|A=a) = P(Ŷ=1\|A=b) |
| Equal opportunity | 24 | P(Ŷ=1\|Y=1,A=a) = P(Ŷ=1\|Y=1,A=b) |
| Equalized odds | 25 | TPR e FPR uguali tra gruppi |
| Disparate impact | 26 | rapporto P(Ŷ=1\|A=a)/P(Ŷ=1\|A=b), regola 80% |
| Predictive parity | 27 | P(Y=1\|Ŷ=1,A=a) = P(Y=1\|Ŷ=1,A=b) |
| Calibration | 28 | P(Y=1\|score, A) uguale tra gruppi |

### Analisi delle feature (slide 29)
- Feature importance
- Correlation analysis
- **SHAP / LIME** — OBBLIGATORIO, è l'aggancio per l'interpretabilità

### Audit algoritmico (slide 30-34) — 6 FASI = STRUTTURA SEZIONE 5
1. Definizione contesto
2. Identificazione feature sensibili
3. Analisi dei dati
4. Valutazione modello
5. Mitigazione bias
6. Documentazione (Model Cards, Datasheets)

### Tecniche di mitigation (slide 35-50) — 1 PER CATEGORIA
**Pre-processing** (slide 36-37):
- Reweighing ← scelto come pre-processing
- Sampling bilanciato (oversampling: SMOTE, ADASYN; undersampling:
  Tomek, ENN, NearMiss)
- Discretizzazione equa
- Fair Representation Learning

**In-processing** (slide 47-48):
- Fairness Constraints (es. Fairlearn ExponentiatedGradient) ← scelto
- Adversarial Debiasing
- Prejudice Remover

**Post-processing** (slide 49-50):
- Equalized Odds Post-processing (Fairlearn ThresholdOptimizer) ← scelto
- Reject Option Classification
- Calibrated Equalized Odds

### Riferimenti dati dal prof (slide 51)
- Barocas, Hardt, Narayanan: *Fairness and Machine Learning*, MIT Press
  2023 — https://fairmlbook.org
- Mehrabi, Morstatter, Saxena, Lerman, Galstyan: *A Survey on Bias and
  Fairness in ML*, ACM Comput. Surv. 54(6) 2022

## Tecniche AMMISSIBILI — modulo Privacy

### Concetti base (slide 10-13)
- Identificatori diretti vs quasi-identificatori — **OBBLIGATORIO**
  identificare entrambi i tipi nel dataset

### Anonimizzazione e pseudo-anonimizzazione (slide 14-22)
- Rimozione, generalizzazione, soppressione, aggregazione
- GDPR Considerando 26 — **OBBLIGATORIO** citare

### k-anonymity (slide 31-44) — IMPLEMENTATA
- Definizione: ogni record indistinguibile da almeno k-1 altri sui QID
- Generalizzazione (gerarchie su attributi)
- Soppressione (rimozione record outlier)
- Approccio iterativo / partizionamento

### l-diversity (slide 45-48) — ACCENNATA + ESEMPIO MINIMALE
- Attacco di omogeneità sulla k-anonymity
- Distinct l-diversity

### t-closeness (slide 49-52) — SOLO CITATA
- Solo menzione concettuale, non implementata

### Differential privacy (slide 23-30) — IMPLEMENTATA
- Definizione ε-DP, ε-δ-DP
- Sensibilità globale L1
- Meccanismo di Laplace ← OBBLIGATORIO usare diffprivlib.Laplace
- Privacy budget e composizione

### Federated learning, SMPC, crittografia omomorfa (slide 53-115)
- **SOLO CITATI nella sezione "estensioni future"**, non implementati.
  Sono concettualmente fuori scope per un singolo dataset tabulare su
  Colab.

### Riferimenti dati dal prof
- Wood, Altman, Bembenek, Bun et al.: *Differential Privacy: A Primer
  for a Non-technical Audience*
- Evans, Kolesnikov, Rosulek: *A Pragmatic Introduction to Secure
  Multi-Party Computation*

## Tecniche NON DA USARE (anche se appetitose)

| Tecnica | Perché no |
|---|---|
| Counterfactual explanations (DiCE) | Non nel syllabus |
| Anchors explanations | Non nel syllabus |
| Causal inference frameworks (DoWhy, EconML) | Non nel syllabus |
| Neural networks / Deep Learning | Overkill, non nel syllabus per ML supervisionato |
| GANs per dati sintetici | Fuori scope |
| Federated learning effettivo | Concettualmente trattato ma impraticabile su single notebook |
| Crittografia omomorfa effettiva | Idem |
| Synthetic data generation con Faker oltre demo | Non rilevante |

## Riferimenti normativi da CITARE nel notebook

| Norma | Articolo | Dove citarlo |
|---|---|---|
| AI Act (Reg. UE 2024/1689) | Art. 10 (dati di training per high-risk AI) | Sezione bias |
| GDPR (Reg. UE 2016/679) | Art. 22 (decisioni automatizzate) | Sezione bias, introduzione |
| GDPR | Considerando 26 (definizione dato anonimo) | Sezione privacy/anonimizzazione |

## Riferimenti filosofici di approfondimento (oltre quelli del prof)

Letture aggiuntive usate per nutrire le sezioni di discussione
(sezioni 9, 13, 14 del notebook). Non sostituiscono i riferimenti
del prof, li integrano sul versante critico/filosofico.

- **Barocas, Hardt, Narayanan**, *Fairness and Machine Learning*,
  MIT Press 2023 — https://fairmlbook.org
  - **Cap. 1 "Why ML?"** — limiti della fairness intesa come
    proprietà puramente tecnica del modello; il bias come fenomeno
    socio-tecnico context-dependent. Aggancio: narrativa CA vs MS
    nel notebook (sezione 2 + discussione integrata sezione 14).
  - **Cap. 3 "Classification"** — formalizzazione delle metriche
    di fairness, impossibility theorem (Chouldechova/Kleinberg),
    trade-off tra demographic parity, equalized odds e calibration.
    Aggancio: sezione 6 (markdown finale sull'impossibility) e
    sezione 9 (curva di Pareto accuracy↔fairness).
