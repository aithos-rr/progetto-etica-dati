# Progetto pratico — Scienza ed etica dei dati

Analisi etica del dataset **ACSIncome** (US Census via `folktables`) 
con focus su **bias ed equità** e **protezione della privacy**, 
realizzata come progetto pratico per l'esame del corso di 
*Scienza ed etica dei dati* (prof. R. Rosati).

**Sapienza Università di Roma** — Corso di laurea in 
Filosofia e Intelligenza Artificiale, a.a. 2025/2026.

**Autori**:
- Riccardo Romano — matricola 2126461
- Hera Polegri — matricola 2153537

---

## Sintesi del progetto

Il notebook applica al dataset ACSIncome (Ding et al., NeurIPS 2021)
un audit etico in due parti, mappato puntualmente sulle slide del 
corso del prof. Rosati:

- **Parte A — Bias ed equità**: EDA quantitativa, modelli baseline 
  (Logistic Regression + Gradient Boosting), audit algoritmico in 6 
  fasi, **6 metriche di fairness**, interpretabilità con SHAP e LIME, 
  **3 tecniche di mitigation** (Reweighing, Exponentiated Gradient, 
  Threshold Optimizer), analisi del trade-off accuracy ↔ fairness.

- **Parte B — Protezione della privacy**: identificazione di quasi-id, 
  applicazione di **k-anonymity** con generalizzazione (5-anonymity 
  raggiunta sul 99.9% dei record), accenno di l-diversity, 
  **differential privacy** con meccanismo di Laplace e analisi del 
  trade-off ε ↔ utility.

- **Sintesi etico-normativa**: discussione del **trilemma 
  bias/privacy/utility**, riferimenti ad **AI Act art. 10**, 
  **GDPR art. 22** e **Considerando 26**.

## Risultati principali

- **Bias rilevato e quantificato**: gender gap di 14.7pp, range 
  razziale di 33pp, gap inter-statale di 14.1pp sul tasso di reddito 
  > 50k.
- **Disparate Impact baseline su SEX = 0.544**, **viola la regola 
  dell'80%**.
- **Reweighing porta il DI a 0.827** (compliance recuperata) con 
  costo accuracy di soli 0.7 punti percentuali.
- **55.8% dei record del dataset originale è univocamente 
  identificabile** sui soli tre quasi-id AGEP, SCHL, OCCP → secondo 
  GDPR Cons. 26 non è anonimo.
- **k-anonymity con generalizzazione** raggiunge k≥5 sul 99.9% dei 
  record sopprimendo solo 21 record (0.1%); l'accuracy del modello 
  ML sui dati anonimizzati cala di soli 0.6 punti percentuali.
- **Differential privacy a ε=1** introduce errore medio di 0.006 
  anni sulla query "AGEP medio per stato" — sotto la soglia 
  di sensibilità umana.

## Struttura del repository
docs/                  documenti di specifica del progetto
PROJECT_BRIEF.md     visione esecutiva e decisioni strategiche
SYLLABUS_MAPPING.md  mapping tecniche del progetto ↔ slide del corso
NOTEBOOK_SPEC.md     specifica strutturale del notebook (16 sezioni)
notebooks/
progetto_etica_dati.ipynb   notebook completo, eseguibile end-to-end
data/                  cartella di destinazione dei dataset scaricati
a runtime da Folktables (gitignored)

## Esecuzione

Il notebook è progettato per girare su **Google Colab** senza 
modifiche né setup locale. Procedura:

1. Aprire [Google Colab](https://colab.research.google.com).
2. File → Apri notebook → GitHub → `aithos-rr/progetto-etica-dati`
   → selezionare `notebooks/progetto_etica_dati.ipynb`.
3. Runtime → Restart and run all.

Tempo di esecuzione completo atteso: ~3-5 minuti su Colab free tier
(la cella più lenta è l'ExponentiatedGradient della Sezione 8, ~60-90s).

### Dipendenze

Tutte le librerie non preinstallate su Colab sono installate 
automaticamente dalla prima cella del notebook:

- `folktables` — accesso al dataset US Census ACS
- `fairlearn` — metriche e mitigazione del bias
- `shap`, `lime` — interpretabilità post-hoc
- `imbalanced-learn` — gestione class imbalance
- `diffprivlib` (IBM) — meccanismi di differential privacy

Le seguenti sono già preinstallate su Colab e usate dal notebook:
`pandas`, `numpy`, `scikit-learn`, `matplotlib`, `seaborn`, `scipy`.

## Riferimenti

Il progetto si appoggia sulle fonti citate nelle slide del corso:

- Solon Barocas, Moritz Hardt, Arvind Narayanan, *Fairness and 
  Machine Learning — Limitations and Opportunities*, MIT Press, 
  2023. <https://fairmlbook.org>
- Ninareh Mehrabi et al., *A Survey on Bias and Fairness in 
  Machine Learning*, ACM Comput. Surv. 54(6):115:1-115:35, 2022.
- Cynthia Dwork & Aaron Roth, *The Algorithmic Foundations of 
  Differential Privacy*, Foundations and Trends in Theoretical 
  Computer Science, 2014.
- Frances Ding, Moritz Hardt, John Miller, Ludwig Schmidt, 
  *Retiring Adult: New Datasets for Fair Machine Learning*, 
  NeurIPS 2021.

## Licenza

Codice rilasciato come progetto universitario per finalità 
didattiche. Il dataset ACSIncome è pubblicamente accessibile 
attraverso la libreria `folktables`, derivato da dati del 
US Census Bureau (Public Use Microdata Sample).
