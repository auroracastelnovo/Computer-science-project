# 🚕 NYC Yellow Taxi Data Analysis — Foundations of Computer Science Final Project

![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458.svg)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626.svg)
![Status](https://img.shields.io/badge/Status-Completed-success.svg)

Progetto finale per il corso di **Foundations of Computer Science** (Laurea Magistrale). Il progetto consiste in un'analisi approfondita, pulizia dei dati e implementazione di un algoritmo di tracciamento di corse concatenate (*Trip Chaining*) sui dati dei **NYC Yellow Taxis (TLC)**.

---

## 👤 Autrice

* **Aurora Castelnovo** (Matricola: 864811)

---

## 📌 Descrizione del Progetto

Il progetto analizza un dataset di corse di taxi di New York City (`data.csv` fornito dalla NYC Taxi and Limousine Commission - TLC). L'obiettivo è rispondere a 17 quesiti specifici legati al caricamento dei dati, data cleaning, aggregazione ed esaustive analisi statistiche e temporali, fino all'implementazione di un algoritmo avanzato basato su `pd.merge_asof` e componenti connesse (*Path Compression*) per ricostruire la "catena" di corse effettuate consecutivamente dai medesimi taxi.

---

## 🛠️ Tecnologie Utilizzate

* **Python 3.x**
* **Pandas**: Manipolazione, aggregazione, pulizia dati, gestione datetime e join asincroni (`pd.merge_asof`).
* **NumPy**: Calcolo vettoriale ed elaborazione numerica.
* **Jupyter Notebook**: Ambiente di sviluppo e presentazione dei dati.

---

## 📊 Fasi dell'Analisi e Struttura del Notebook

### 1. Ingestione e Pulizia dei Dati (Data Cleaning & Preprocessing)
* **Tipi di dato ottimizzati**: Uso di tipi integer annullabili (`Int64`) per colonne come `VendorID`, `RatecodeID`, `passenger_count`, `payment_type` per gestire correttamente i valori missing (`NaN`).
* **Parsing Datetime**: Conversione automatica delle colonne temporali (`tpep_pickup_datetime`, `tpep_dropoff_datetime`).
* **Gestione valori mancanti**: Imputazione di `store_and_fwd_flag` con `'Undefined'`.
* **Rimozione anomalie temporali**: Eliminazione delle corse in cui l'orario di partenza è successivo all'orario di arrivo.

---

### 2. Quesiti e Milestone del Progetto (1 - 17)

| # | Quesito / Task | Descrizione dell'Approccio Metodologico |
|---|---|---|
| **1** | **Filtro corse lunghe** | Estrazione di tutte le corse con `trip_distance > 50` ed eliminazione degli outlier estremi (es. `trip_distance > 210,000`). |
| **2** | **Pagamenti mancanti** | Estrazione delle corse con `payment_type` nullo (`isnull()`). |
| **3** | **Conteggio per coppia (Pickup, Dropoff)** | Raggruppamento tramite `groupby(['PULocationID', 'DOLocationID'])` e calcolo frequenza con `.size()`. |
| **4** | **Gestione dati non validi (`bad` dataframe)** | Isolamento delle righe contenenti valori `NaN` in colonne critiche in un dataframe chiamato `bad`, e successiva eliminazione dal dataset principale `nyc`. |
| **5** | **Calcolo durata della corsa** | Creazione della colonna `duration` in minuti calcolata come `(dropoff - pickup).dt.total_seconds() / 60`. |
| **6** | **Corse per punto di partenza** | Calcolo delle corse totali iniziate in ciascuna `PULocationID`. |
| **7** | **Clustering orario a 30 minuti** | Suddivisione della giornata in 48 intervalli da 30 minuti (`cluster = (hour * 60 + minute + second/60) // 30`). |
| **8** | **Passeggeri e tariffe medie per intervallo** | Calcolo della media (`mean()`) di `passenger_count` e `fare_amount` per ogni cluster temporale. |
| **9** | **Tariffa media per tipo pagamento e intervallo** | Raggruppamento per `(payment_type, cluster)` e calcolo di `fare_amount.mean()`. |
| **-** | *Trattamento rimborsi/cancellazioni* | Identificazione delle corse con valori monetari negativi, conversione in positivo e rimozione delle coppie duplicate (cancellazioni), seguita dall'eliminazione delle righe negative residue (`nyc_new`). |
| **10** | **Massima tariffa media per tipo pagamento** | Individuazione del cluster orario con il `fare_amount` medio più elevato per ogni tipo di pagamento (`idxmax()`). |
| **11** | **Massimo rapporto Mancia / Tariffa** | Calcolo del rapporto `tip_sum / fare_sum` per ciascun tipo di pagamento e cluster, identificando l'intervallo con il rapporto massimo. |
| **12** | **Location con tariffa media più alta** | Determinazione della `PULocationID` e della `DOLocationID` caratterizzate dalla tariffa media più elevata. |
| **13** | **Dataframe `common` (Top 5 destinazioni)** | Filtraggio del dataset mantenendo per ogni `PULocationID` solo le 5 destinazioni (`DOLocationID`) più frequenti. |
| **14** | **Tariffa media sul dataframe `common`** | Ricalcolo della tariffa media per `(payment_type, cluster)` limitatamente alle tratte più comuni. |
| **15** | **Differenza di tariffa media** | Calcolo della differenza assoluta tra le tariffe medie su `common` e quelle del dataset generale (`diff = fare_common - fare_nyc`). |
| **16** | **Rapporto di variazione relativa** | Calcolo della variazione percentuale/proporzionale delle tariffe (`diff / fare_nyc`). |
| **17** | **Algoritmo di Trip Chaining** | Ricostruzione delle catene di corse consecutive effettuate dallo stesso taxi (`VendorID`) in cui il pickup della corsa $N+1$ avviene nella stessa location del dropoff della corsa $N$ entro **2 minuti**. Utilizzo di `pd.merge_asof`, filtraggio duplicati e algoritmo di **Path Compression** per unire le componenti connesse del grafo. |

---

## 🔬 Dettaglio Algoritmo "Trip Chaining" (Punto 17)

L'algoritmo per la costruzione delle catene di viaggi risolve un problema di tracciamento sequenziale su grandi moli di dati:
1. **As-of Merge temporale**: Sfrutta `pd.merge_asof` con tolleranza `tolerance=pd.Timedelta('2 minutes')`, `direction='forward'` associando l'orario di dropoff di un viaggio con l'orario di pickup successivo a parità di `VendorID` e `location`.
2. **Disambiguazione**: Mantiene solo il primo evento sequenziale valido per evitare cicli o biforcazioni multiple.
3. **Graph Union-Find & Path Compression**: Assegna a ciascuna corsa un ID iniziale di catena coincidente con il suo indice, aggiorna le relazioni in un dizionario e risale all'antenato comune radice (*ultimate root*) tramite una ricerca iterativa, raggruppando l'intera sequenza di corse sotto un unico identificatore univoco `chain`.

---

## 🚀 Come Eseguire il Notebook

### Pre-requisiti
Assicurati di aver installato Python 3.8+ e i seguenti pacchetti:
```bash
pip install pandas numpy jupyter
```

### Istruzioni
1. Clona la repository o scarica i file di progetto:
   ```bash
   git clone https://github.com/tuo-username/NYC-Taxi-Data-Analysis.git
   cd NYC-Taxi-Data-Analysis
   ```
2. Inserisci il file di dati `data.csv` nella directory corretta o aggiorna il percorso nel notebook:
   ```python
   nyc = pd.read_csv('data.csv', low_memory=False, dtype=..., parse_dates=...)
   ```
3. Avvia Jupyter Notebook:
   ```bash
   jupyter notebook "Final project AFN.ipynb"
   ```
4. Esegui le celle in sequenza.

---

## 📄 Licenza

Questo progetto è sviluppato a fini accademici per il corso di **Foundations of Computer Science**. Tutti i dati utilizzati provengono dal dataset pubblico **NYC Taxi & Limousine Commission (TLC)**.
