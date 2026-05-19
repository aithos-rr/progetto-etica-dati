# NOTEBOOK SPEC — Struttura e criteri di accettazione

Questo documento specifica **cosa va in ogni sezione del notebook**, in
che ordine, e con quali criteri di accettazione. Sostituisce ogni
discussione di scope: se non è qui, non va nel notebook.

## Convenzioni

- **`[CODE]`** = cella di codice (gestita con Claude Code)
- **`[MD]`** = cella markdown (gestita in chat con Claude orchestratore)
- **`[CODE+MD]`** = sezione mista, va sviluppata insieme
- **DoD** = Definition of Done (criterio di accettazione)

## Struttura macro (16 sezioni)

```
0. Setup ambiente Colab                                     [CODE]
1. Introduzione e framing etico                             [MD]
2. Dataset: descrizione, sensibili, quasi-identificatori    [CODE+MD]
3. EDA quantitativa                                         [CODE]
4. Modelli baseline                                         [CODE]
═══ PARTE A — BIAS ED EQUITÀ ═══════════════════════════════
5. Audit algoritmico (6 fasi del prof)                      [CODE+MD]
6. Metriche di fairness (tutte e 6)                         [CODE]
7. Spiegabilità con SHAP e LIME                             [CODE]
8. Mitigation (Reweighing, ExpGrad, ThresholdOptimizer)     [CODE]
9. Trade-off accuracy ↔ fairness                            [CODE+MD]
═══ PARTE B — PRIVACY ══════════════════════════════════════
10. Identificatori e quasi-identificatori nel dataset       [CODE+MD]
11. k-anonymity e accenno l-diversity                       [CODE]
12. Differential privacy                                    [CODE]
13. Trade-off privacy ↔ utility                             [CODE+MD]
═══ SINTESI ════════════════════════════════════════════════
14. Discussione integrata: trilemma bias/privacy/utility    [MD]
15. Riferimenti normativi                                   [MD]
16. Limiti e conclusioni                                    [MD]
```

---

## Sezione 0 — Setup ambiente Colab `[CODE]`

**Contenuto**:
- Cella che installa tutte le librerie (`!pip install ...`)
- Cella che importa tutto e fissa i seed (`np.random.seed(42)`,
  `random.seed(42)`)
- Stampa versione librerie principali per riproducibilità

**Librerie**: `folktables, pandas, numpy, scikit-learn, fairlearn,
shap, lime, imbalanced-learn, diffprivlib, matplotlib, seaborn, scipy`

**DoD**:
- Esecuzione `pip install` riuscita su Colab free in <2 minuti
- Tutti gli import funzionano senza warning critici
- Stampa la versione di Python e delle 5 librerie principali

---

## Sezione 1 — Introduzione e framing etico `[MD]`

**Contenuto**: 200-300 parole. Risponde a:
- Perché il problema "predire il reddito" è eticamente carico
- Cosa intendiamo per "etica dei dati e dei modelli" nel progetto
- Quali aree etiche affrontiamo (Bias + Privacy) e perché queste due
- Riferimento esplicito al syllabus del corso

**Riferimenti obbligatori**: GDPR art. 22 (decisioni automatizzate)

**DoD**:
- Non è generata da LLM in stile generico
- Cita esplicitamente lo scope del progetto (Bias + Privacy, niente
  Autenticità, niente Federated Learning)
- 200-300 parole

---

## Sezione 2 — Dataset: descrizione e attributi `[CODE+MD]`

**Markdown** (~150 parole):
- Cos'è ACSIncome / Folktables (Ding, Hardt, Miller, Schmidt 2021)
- Perché 2 stati: California (diverso da Mississippi)
- Lista esplicita degli attributi: target, sensibili, quasi-ID

**Codice** — procedura corretta in 5 step (necessaria perché
`ACSIncome.df_to_pandas()` scarta la colonna `ST` dello stato, che
invece serve alla narrativa CA vs MS):
1. Download separato dei due stati via
   `data_source.get_data(states=["CA"])` e
   `data_source.get_data(states=["MS"])`
2. Per ogni stato, applicare `ACSIncome.df_to_pandas(data)` per
   ottenere `(features, label, group)`
3. Aggiungere a mano una colonna `STATE` al DataFrame `features`
   con valore costante `'CA'` o `'MS'`
4. Concatenare i due DataFrames `features` e i due array `label`
5. **Subsample stratificato** sulla colonna `STATE` finale →
   50.000 record totali (25k CA + 25k MS), `random_state=42`

Dopo il subsample:
- Display di `df.head()`, `df.shape`, `df.dtypes`
- Tabella che classifica ogni colonna come: target / sensibile /
  quasi-identificatore / feature normale

**DoD**:
- Dataset scaricato e caricato su Colab in <60 secondi
- Dimensione finale: **50.000 record bilanciati 25k+25k tra CA e MS
  via stratified sampling** (`random_state=42`)
- Colonna `STATE` presente e correttamente popolata (`'CA'`/`'MS'`)
- Classificazione attributi visualizzata in formato leggibile

---

## Sezione 3 — EDA quantitativa `[CODE]`

**Mappa al syllabus**: bias slide 19-21 + analisi-dati slide 10-25

**Contenuto**:
1. Distribuzione frequenze del target (esito positivo > $50k)
2. Distribuzione frequenze attributi sensibili (race, sex)
3. **Distribuzione condizionata**: % esito positivo per ogni gruppo
   di race × sex × STATE
4. Visualizzazioni comparative: bar plot, boxplot di age per gruppi
5. **Test statistici** (slide bias 21):
   - Chi-quadro su (race, target)
   - Chi-quadro su (sex, target)
   - KS test su distribuzione age tra esiti positivi/negativi
6. Heatmap correlazioni numeriche

**DoD**:
- Tutti i plot hanno titolo, label assi, legenda
- Test statistici stampano p-value con interpretazione
- Sotto ogni plot/test, 1-2 righe di commento esplicativo (cella
  markdown)

---

## Sezione 4 — Modelli baseline `[CODE]`

**Mappa al syllabus**: analisi-dati slide 60-69

**Modelli**:
1. Logistic Regression (interpretabile by design)
2. Random Forest o Gradient Boosting (più potente, meno interpretabile)

**Pipeline**:
- Train/test split stratificato (test_size=0.3, random_state=42)
- Preprocessing: encoding categoriche, scaling numeriche
- Training entrambi i modelli
- Valutazione: accuracy, precision, recall, F1, matrice di confusione

**DoD**:
- Accuracy baseline ragionevole (>0.75 atteso su ACSIncome)
- Matrice di confusione visualizzata per entrambi i modelli
- Salvataggio modelli in memoria per riuso nelle sezioni successive

---

## Sezione 5 — Audit algoritmico `[CODE+MD]`

**Mappa al syllabus**: bias slide 30-34 (le 6 fasi)

**Struttura ESPLICITA** in 6 sotto-sezioni che ricalcano le slide:

### 5.1 Definizione del contesto `[MD]`
- Obiettivo: predire reddito >$50k
- Decisione automatizzata: scenario ipotetico (es. approvazione credito)
- Gruppi potenzialmente impattati: minoranze razziali, donne

### 5.2 Identificazione feature sensibili `[MD]`
- Sensibili dirette: RACE (codici 1-9), SEX (1=maschio, 2=femmina)
- **Possibili proxy**: POBP (place of birth), NATIVITY → discutere

### 5.3 Analisi dei dati `[CODE]`
- Rappresentatività dei gruppi (riprende sezione 3)
- Correlazioni feature sensibili ↔ target
- Class imbalance per gruppo

### 5.4 Valutazione del modello `[CODE]`
- Accuracy per sottogruppo (race × sex × STATE)
- Confusion matrix per sottogruppo principale
- Setup per metriche di fairness (sezione 6)

### 5.5 Mitigazione del bias `[MD - placeholder]`
- Anticipa la sezione 8

### 5.6 Documentazione e trasparenza `[MD - placeholder]`
- Anticipa: Model Card alla fine del notebook

**DoD**:
- Ogni sotto-sezione esiste come cella separata con titolo
- Le 6 fasi sono esplicitamente numerate e nominate come nelle slide

---

## Sezione 6 — Metriche di fairness `[CODE]`

**Mappa al syllabus**: bias slide 22-28 (TUTTE E 6)

**Implementazione**:
1. **Demographic parity**: differenza P(Ŷ=1) tra gruppi
2. **Equal opportunity**: differenza TPR tra gruppi
3. **Equalized odds**: differenza TPR + FPR tra gruppi
4. **Disparate impact**: rapporto P(Ŷ=1|A=a)/P(Ŷ=1|A=b), regola 80%
5. **Predictive parity**: differenza precision (PPV) tra gruppi
6. **Calibration**: reliability diagram per gruppo

**Librerie**: usare `fairlearn.metrics` (MetricFrame) come ossatura,
calcolare a mano le metriche che non sono in libreria.

**Output**:
- Tabella riassuntiva tutte e 6 le metriche, per race e per sex
- Identificazione delle metriche che falliscono (DI < 0.8 o > 1.25,
  differenze >10%)
- Plot a barre comparativo

**Discussione**: 1 cella markdown alla fine che spiega
**l'impossibility theorem** (Chouldechova/Kleinberg): non si possono
soddisfare contemporaneamente DP, equalized odds e calibration. Cita
Barocas-Hardt-Narayanan cap. 3.

**DoD**:
- Tutte e 6 le metriche calcolate per entrambi gli attributi sensibili
- Almeno 2 metriche mostrano violazione della fairness sul baseline
- Markdown finale sull'impossibility theorem, 5-8 righe

---

## Sezione 7 — Spiegabilità SHAP e LIME `[CODE]`

**Mappa al syllabus**: bias slide 29

**Contenuto**:
1. **SHAP globale**: feature importance + summary plot per modello
   Gradient Boosting
2. **SHAP locale**: waterfall plot per 3 casi specifici:
   - Caso A: persona predetta correttamente positiva
   - Caso B: persona predetta correttamente negativa
   - Caso C: persona al confine (probabilità ~0.5) o falsa positiva
3. **LIME**: spiegazione locale sugli stessi 3 casi
4. **Confronto SHAP vs LIME**: SHAP e LIME concordano? Quando
   divergono?

**DoD**:
- 1 summary plot SHAP globale leggibile
- 3 waterfall plot SHAP locali
- 3 explanation LIME
- 1 markdown finale di confronto, 5-10 righe

---

## Sezione 8 — Mitigation `[CODE]`

**Mappa al syllabus**: bias slide 35-50

**3 tecniche, una per categoria**:

### 8.1 Pre-processing: Reweighing
- Formula esplicita: w(s,y) = P(s)·P(y) / P(s,y) — slide 37
- Implementazione manuale (no AIF360, che è pesante su Colab) +
  sample_weights in `model.fit()`
- Rivaluta metriche di fairness

### 8.2 In-processing: Fairlearn ExponentiatedGradient
- Constraint: `DemographicParity` o `EqualizedOdds`
- Training del modello "fairness-aware"
- Rivaluta metriche

### 8.3 Post-processing: Fairlearn ThresholdOptimizer
- Constraint: `equalized_odds`
- Applicato sopra il modello baseline
- Rivaluta metriche

**Tabella finale**: 4 colonne (baseline, reweighing, exp_grad,
threshold), 6 righe (le 6 metriche di fairness) + 1 riga (accuracy).

**DoD**:
- Tutte e 3 le tecniche implementate
- Tabella confronto leggibile
- Differenze visibili tra baseline e mitigated

---

## Sezione 9 — Trade-off accuracy ↔ fairness `[CODE+MD]`

**Codice**:
- Plot scatter con asse X = accuracy, asse Y = una metrica di fairness
  (es. DP difference o |1 - DI|)
- 4 punti: baseline + 3 mitigated

**Markdown** (~200 parole):
- Discussione: c'è una "curva di Pareto"?
- Quale tecnica vince in base al criterio scelto?
- Riferimento all'AI Act art. 10 (qualità dati per high-risk AI)

**DoD**:
- Plot leggibile
- Discussione filosofica originale, non generica

---

## Sezione 10 — Identificatori e quasi-id `[CODE+MD]`

**Mappa al syllabus**: privacy slide 10-13

**Markdown** (~150 parole):
- Definizione identificatore diretto vs quasi-identificatore
- Citare GDPR Considerando 26

**Codice**:
- Nel dataset ACSIncome NON ci sono identificatori diretti (è già
  anonimizzato dal Census)
- Quasi-identificatori potenziali: AGEP (age), SCHL (education), OCCP
  (occupation), POBP (place of birth), MAR (marital status)
- Calcolo iniziale di k-anonymity sui QID scelti: per ogni combinazione
  di valori, quanti record? Distribuzione del k.

**DoD**:
- Visualizzazione distribuzione k iniziale (es. istogramma o tabella)
- Identificazione record con k=1 (unici, "fingerprint")

---

## Sezione 11 — k-anonymity e l-diversity `[CODE]`

**Mappa al syllabus**: privacy slide 31-48

### 11.1 k-anonymity con generalizzazione
- Generalizzazione AGEP in bin (es. 18-25, 26-35, 36-50, 51+)
- Generalizzazione SCHL in macro-categorie (no edu / high school /
  college / graduate)
- Generalizzazione OCCP in macro-categorie (white collar / blue collar
  / service / other)
- Verifica raggiungimento di k=5 (soppressione record residui)
- Confronto: % record persi per soppressione

### 11.2 Accenno l-diversity
- Sui gruppi k-anonimi, verifica diversità del target reddito
- Identifica eventuali gruppi che soddisfano k-anonymity ma falliscono
  l-diversity con l=2

### 11.3 t-closeness (solo menzione)
- Markdown di 3 righe: definizione + perché non implementiamo

**DoD**:
- Dataset finale 5-anonimo prodotto
- Distribuzione record persi quantificata
- Almeno un esempio di attacco di omogeneità identificato

---

## Sezione 12 — Differential Privacy `[CODE]`

**Mappa al syllabus**: privacy slide 23-30

**Contenuto**:
1. **Setup**: dataset originale, query: età media per stato
2. **Senza DP**: query diretta su dataset originale
3. **Con DP**: meccanismo di Laplace via `diffprivlib.mechanisms.Laplace`
   - Sensibilità globale L1 calcolata esplicitamente
   - 3 valori di ε: {0.1, 1, 10}
   - Per ogni ε, 100 run → distribuzione dei risultati
4. **Plot**: 3 istogrammi sovrapposti dei 100 risultati per i 3 ε
5. **Privacy budget**: discussione concettuale della composizione (5
   righe)

**DoD**:
- Plot mostra chiaramente che ε piccolo → più rumore, ε grande → meno
- Riferimento esplicito alle slide del prof (formula sensibilità,
  meccanismo Laplace)

---

## Sezione 13 — Trade-off privacy ↔ utility `[CODE+MD]`

**Codice**:
- Plot: asse X = ε, asse Y = errore medio della query rispetto al vero
- Curva con 5+ punti di ε

**Markdown** (~200 parole):
- Discussione: come si sceglie ε in pratica?
- Riferimento normativo: GDPR Considerando 26 (criterio di
  "ragionevole probabilità" di identificazione)

**DoD**:
- Curva monotona decrescente (più privacy = più errore)
- Discussione filosofica originale

---

## Sezione 14 — Discussione integrata `[MD]`

**Contenuto** (~400-500 parole): il pezzo più importante del progetto.

Tema: **il trilemma bias / privacy / utility**.

- Cosa abbiamo trovato applicando le tecniche su ACSIncome
- Le tecniche di mitigation del bias riducono l'accuracy
- Le tecniche di privacy (k-anonymity, DP) riducono l'utility
- Esiste una soluzione che li ottimizzi tutti contemporaneamente?
- Riferimento all'impossibility theorem (sezione 6)
- Tesi originale dello studente: come dovremmo posizionarci tra
  queste tre tensioni? Quale prevale? In quale contesto?

**Questo va scritto in chat con Claude, non con LLM in batch.**

**DoD**:
- 400-500 parole
- Argomentazione personale, non manualistica
- Riferimenti espliciti alle sezioni precedenti del notebook

---

## Sezione 15 — Riferimenti normativi `[MD]`

**Contenuto** (~250 parole):
- **AI Act (Reg. UE 2024/1689) art. 10**: requisiti per dati di
  training di sistemi AI ad alto rischio (qualità, rappresentatività,
  identificazione bias)
- **GDPR art. 22**: diritto a non essere soggetti a decisioni
  esclusivamente automatizzate
- **GDPR Considerando 26**: criterio per definire "dato anonimo" (mezzi
  ragionevolmente utilizzabili per re-identificare)

Per ognuno: 1 paragrafo che spiega come si collega al notebook.

**DoD**:
- Tre norme citate con riferimento esatto
- Collegamento ai risultati del notebook esplicitato

---

## Sezione 16 — Limiti e conclusioni `[MD]`

**Contenuto** (~300 parole):

### Limiti del lavoro
- Dataset US, non italiano/europeo: trasferibilità limitata
- 2 stati su 50: campione ridotto
- Non implementati: t-closeness, federated learning, SMPC,
  crittografia omomorfa
- Solo 1 task (classificazione binaria)
- Solo attributi sensibili razza e sesso, non age o disability

### Possibili estensioni
- Replicare su EU statistics (es. Eurostat EU-SILC)
- Implementare federated learning con flwr
- Affrontare autenticità con dataset di immagini sintetiche

### Model Card / Datasheet
- Tabella riassuntiva stile Model Card (slide bias 34): modello,
  scopo, dati, metriche, limiti, mitigation applicate

**DoD**:
- Limiti onestamente riconosciuti
- Model Card presente come tabella finale

---

## Workflow di sviluppo per sezione

Per ogni sezione, il giro è:

1. **In chat**: Claude orchestratore produce un **prompt strutturato
   per Claude Code** che include: riferimento alla sezione di questa
   spec, riferimento al syllabus, vincoli, esempio output atteso.
2. **In VS Code con Claude Code**: lo studente passa il prompt, CC
   genera le celle, lo studente le esegue su Colab.
3. **Back in chat**: lo studente incolla output (codice + risultati
   esecuzione), Claude reviewa e produce feedback.
4. **Refinement**: lo studente applica i fix (a mano o via CC).
5. **Commit Git**: messaggio del tipo `feat: section N - description`.
6. **Aggiornamento**: questo file viene aggiornato per riflettere lo
   stato (✅ done / 🔄 in progress / ⬜ todo).

## Stato attuale

```
0. Setup                                                    ⬜
1. Introduzione                                             ⬜
2. Dataset                                                  ⬜
3. EDA                                                      ⬜
4. Baseline                                                 ⬜
5. Audit                                                    ⬜
6. Metriche fairness                                        ⬜
7. SHAP/LIME                                                ⬜
8. Mitigation                                               ⬜
9. Trade-off bias                                           ⬜
10. Identificatori                                          ⬜
11. k-anonymity                                             ⬜
12. Differential privacy                                    ⬜
13. Trade-off privacy                                       ⬜
14. Discussione integrata                                   ⬜
15. Normativa                                               ⬜
16. Conclusioni                                             ⬜
```
