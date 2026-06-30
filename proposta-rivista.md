
## 1. Descrizione del problema

L'obiettivo del progetto è stimare l'età dei soggetti a partire da segnali biometrici, studiando le
variazioni morfologiche legate all'invecchiamento biologico. Affronteremo il problema come un
compito di **regressione**, ovvero la stima dell'età esatta (in anni), senza far la classificazione.  

Lavoreremo su **due modalità biometriche**, FingerVein e PPG, e su una loro combinazione ottenuta
**accoppiando i campioni in base a sesso ed età**, così da poter verificare se l'integrazione delle
informazioni provenienti da fonti diverse migliori la stima rispetto alle singole modalità.

## 2. Dataset utilizzati

Come concordato, **non utilizzeremo dataset pubblici**, così da evitare la variabilità e i problemi metodologici legati all'uso di sensori di acquisizione differenti. 

Definiamo tre configurazioni:
1. **Dataset 1: FingerVein:** dati di vena del dito.
2. **Dataset 2: PPG:** segnali fotopletismografici.
3. **Dataset combinato (FingerVein + PPG):** ottenuto accoppiando campioni FingerVein e PPG con lo stesso
   sesso e la stessa età.

Le due modalità hanno natura diversa e quindi richiedono pipeline e architetture distinte:

- **FingerVein:** **immagini 2D (almeno crediamo)** del pattern venoso del dito;
- **PPG:** **segnale 1D** nel tempo.

**Preprocessing.** Useremo due pipeline dedicate.

- *PPG (1D):* ricampionamento, suddivisione in finestre temporali a lunghezza fissa, rimozione del
  rumore (filtraggio) e normalizzazione.
- *FingerVein (immagini 2D):* estrazione della ROI, normalizzazione di illuminazione/contrasto,
  ridimensionamento a risoluzione fissa, normalizzazione dei pixel ed eventuale data augmentation
  (ad esempio con piccole rotazioni/traslazioni) per limitare l'overfitting.

Studieremo in entrambi i casi le **variazioni morfologiche** legate all'invecchiamento: la morfologia
d'onda per il PPG e la texture/struttura del pattern venoso per il FingerVein.

**Data splitting (rigorosamente per soggetto).** La divisione in Training / Validation / Test sarà
effettuata **per soggetto** all'interno di ciascuna modalità: i dati di una persona presenti nel train non
compariranno mai in validation o test. L'accoppiamento per sesso ed età per il dataset combinato verrà
effettuato **dopo** la suddivisione, così da evitare qualsiasi leakage tra train, validation e test.

## 3. Possibili soluzioni

Addestreremo **tre modelli di Deep Learning**, ciascuno sui propri dati (nessun test cross-modale, dato
che FingerVein e PPG hanno natura e dimensionalità diverse e non sono interscambiabili in input):

- **Modello A:  FingerVein**, addestrato e valutato su Dataset 1;
- **Modello B:  PPG**, addestrato e valutato su Dataset 2;
- **Modello C:  multimodale congiunto**, addestrato e valutato sul Dataset combinato.

- **Modello A (FingerVein):** **CNN 2D** sulle immagini del pattern venoso (eventualmente con
  transfer learning da reti pre-addestrate);
- **Modello B (PPG):** **1D-CNN** ed eventualmente architetture **ricorrenti/temporali** (LSTM/GRU)
  per catturare la morfologia d'onda;
- **Modello C (congiunto):** rete a **due rami**: un ramo CNN 2D per il FingerVein e un ramo 1D per il
  PPG, le cui rappresentazioni (embedding) vengono concatenate prima delle teste finali, ovvero una
  **fusione a livello di feature (early fusion)**.

Ogni modello produce in uscita un singolo valore continuo (l'età stimata in anni).

**Esperimento principale (fusione e confronto).** Come da Sua indicazione, l'aspetto più interessante è
il confronto tra due strategie di integrazione delle due modalità:

- **score-level fusion:** combinazione delle uscite dei Modelli A e B (es. media/ media
  pesata sulle stime di età, o regola di fusione ottimizzata in validation);
- **Modello congiunto:** il Modello C addestrato direttamente sul Dataset combinato.

Confronteremo quindi: **A da solo**, **B da solo**, **fusione(A, B)** e **C**, per capire (i) se la
multimodalità batte le singole modalità e (ii) se conviene fondere a livello di punteggio modelli
separati o addestrare un unico modello sui dati accoppiati.

**Suddivisione del lavoro (3 membri).**

- *Lavoro comune:* reperimento dei dati, accoppiamento per sesso ed età dei campioni, pipeline di
  preprocessing, definizione degli split per soggetto e dell'infrastruttura di valutazione (metriche,
  error analysis) condivisa.
- *Sviluppo individuale:* un membro per il Modello A (FingerVein), uno per il Modello B (PPG), uno per
  il Modello C (multimodale congiunto) e per l'implementazione della fusione score-level. Tutti i modelli
  vengono valutati sugli **stessi split** e con le **stesse metriche**, così il confronto è omogeneo.

## 4. Metriche di valutazione

Trattandosi di un problema di **regressione**, le metriche scelte sono quelle adottate nei paper di
riferimento da Lei forniti.

- **MAE**: Mean Absolute Error
- **RMSE**: Root Mean Squared Error
- **r di Pearson**: tra età reale e stimata
- **R²**: coefficiente di determinazione

**Error analysis (come richiesto nella consegna):** analisi dell'errore in funzione dell'età (per
individuare eventuali bias verso giovani/anziani), grafico di **Bland–Altman** tra età reale e stimata
 ed esempi di casi con errore elevato.

Una domanda sui dati. Al momento non ci risultano ancora disponibili i dataset FingerVein e PPG necessari per lo svolgimento del progetto. Le chiediamo cortesemente se fossero già stati condivisi oppure, in caso contrario, se sia possibile avere indicazioni su come accedervi (ad esempio eventuali dataset del Dipartimento o dati raccolti in precedenza ).

