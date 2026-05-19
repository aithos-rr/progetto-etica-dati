# PROJECT BRIEF — Progetto pratico "Scienza ed etica dei dati"

## Identità del progetto

- **Corso**: Scienza ed etica dei dati (prof. Riccardo Rosati)
- **Corso di laurea**: Filosofia e intelligenza artificiale — Sapienza
- **A.A.**: 2025/2026
- **Studente**: Riccardo (singolo, formalmente gruppo di 2)
- **Target voto**: 10-12 su 12 (esonero parziale prova scritta richiede ≥7)

## Obiettivo

Analizzare un dataset reale applicando tecniche etiche dei dati e dei
modelli di AI viste a lezione, in particolare nelle aree di **Bias ed
equità** e **Protezione della privacy** (con interpretabilità SHAP
integrata nell'audit di bias). Il deliverable è un Jupyter Notebook
(`.ipynb`) eseguibile end-to-end su Google Colab.

## Decisioni strategiche già prese

- **Dataset**: ACSIncome (US Census Bureau via libreria `folktables`)
  - Task: classificazione binaria reddito > $50k
  - Attributi sensibili: race (primario), sex (secondario)
  - Quasi-identificatori per modulo privacy: age, SCHL (education), OCCP
    (occupation), POBP (place of birth)
  - Filtro: 2 stati USA, California (CA) + Mississippi (MS)
  - Subsample finale: **50.000 record totali**, ottenuti via
    stratified sampling sulla colonna `STATE` (25k CA + 25k MS),
    `random_state=42`
  - Motivazione narrativa: confronto **context-dependent fairness**
    tra uno stato ad alto reddito medio (CA) e uno a basso reddito
    medio (MS) → il bias rilevato non è proprietà solo del
    modello/dato ma anche del contesto socio-economico. Riferimento:
    Barocas-Hardt-Narayanan cap. 1 sui limiti della fairness come
    proprietà puramente tecnica.
  - **Nota tecnica gestione colonna STATE**: `ACSIncome.df_to_pandas()`
    rimuove la colonna `ST` perché non è feature del task. Per
    preservare lo stato come variabile esplicita nella narrativa CA
    vs MS, scaricare CA e MS separatamente, applicare
    `df_to_pandas()` su ciascuno, aggiungere a mano la colonna
    `STATE` (valore costante `'CA'` o `'MS'`), concatenare, infine
    applicare lo stratified subsample.
- **Aree etiche affrontate**: 2 di 4 (Bias/Equità + Privacy).
  Interpretabilità è sotto-tema del modulo Bias (slide 29 del prof).
  Autenticità non affrontata: fuori scope.
- **Librerie principali**:
  `pandas, scikit-learn, fairlearn, shap, lime, imbalanced-learn,
  folktables, diffprivlib, matplotlib, seaborn`
- **Vincolo HARD**: tutto eseguibile su Colab free tier. Nessuna GPU
  richiesta. Tempo di esecuzione completo del notebook < 10 minuti.

## Vincoli del corso (non negoziabili)

- Output finale: `.ipynb` Jupyter Notebook
- Eseguibile su Google Colab senza modifiche
- Mail al prof Rosati con link al notebook, almeno 7 giorni prima
  dell'appello d'esame
- La presentazione (slide separate) verrà costruita in un secondo
  momento, dopo la consegna del notebook
- Tecniche da usare: "principalmente quelle viste a lezione". Tutto il
  resto è opzionale ma deve essere riconducibile al syllabus.

## Tempistica

- **Sprint**: 7-10 giorni a 6-8 ore/giorno
- **Inizio**: 19 maggio 2026
- **Deadline notebook**: 28 maggio 2026 (margine 1-2 giorni)
- **Deadline mail al prof**: 7 giorni prima appello scelto

## Divisione del lavoro

Lavoro **singolo** (Riccardo gestisce entrambe le voci della coppia).
Workflow operativo:

1. **Chat con Claude (orchestratore)** — pianificazione, design,
   review, parte filosofica
2. **Claude Code in VS Code (executor)** — generazione celle codice
   secondo specifiche
3. **Loop di review**: ogni sezione passa da CC → output a Claude in
   chat → feedback → refinement → commit

## Criteri di qualità

Un notebook punta a 11-12 se rispetta tutti questi criteri:

1. **Mapping puntuale al syllabus** — ogni sezione tecnica dichiara
   esplicitamente a quale slide del prof si riferisce (vedi
   `SYLLABUS_MAPPING.md`)
2. **Narrativa coerente** — il notebook racconta una storia, non è una
   lista di tecniche
3. **Riflessione filosofica originale** nelle sezioni di discussione,
   non commenti generici
4. **Trade-off espliciti** — accuracy↔fairness e privacy↔utility con
   numeri e commento
5. **Riproducibilità** — seed fissati, requirements documentati, run
   end-to-end pulito
6. **Riferimenti normativi** — AI Act art. 10, GDPR art. 22, GDPR
   Cons. 26, almeno citati nel posto giusto

## Anti-pattern da evitare

- Usare tecniche non viste a lezione senza giustificazione esplicita
  (es. counterfactual explanations DiCE, contestual bandits, ecc.)
- Notebook lunghi >40 celle: il prof non lo legge tutto
- Plot ornamentali senza commento
- Sezioni di markdown filosofico generate da LLM in stile generico
  ("L'etica dell'IA è un tema importante perché...")
- Caricare il dataset da remoto senza fallback se l'API è giù
- Dimenticare di citare gli autori (Barocas-Hardt-Narayanan e Mehrabi
  et al. — riferimenti dati dal prof)
