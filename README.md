## 🚀 Obiettivo del Progetto

Il progetto **Archivio-Semantico-AI** automatizza l’analisi, la classificazione e la migrazione di grandi archivi documentali (≈200.000 file) in un sistema **MinIO conforme**, distinguendo **atti ufficiali** da **bozze tecniche**.

L’analisi è ibrida (OCR + AI semantica) e sfrutta:
- **SSD NVMe** per I/O ad alte prestazioni  
- **GPU NVIDIA (es. RTX 5090)** per OCR e inferenze  
- **LLM (Gemini/GPT-4o)** per la classificazione semantica  
- **MinIO** per l’archiviazione finale, con tagging e policy di retention  


# 📁 Struttura del Repository

```
Archivio-Semantico-AI/
│
├── 📄 README.md                     # Documentazione principale del progetto
├── 📄 .env.example                  # Template delle variabili d'ambiente (senza dati sensibili)
├── 📄 requirements.txt              # Dipendenze Python
├── 📄 LICENSE                       # Licenza (es. MIT)
├── 📄 .gitignore                    # Esclusioni Git
│
├── 📂 scripts/                      # Script di automazione e orchestrazione
│   ├── ingest_data.py              # Fase 1: indicizzazione e preparazione CSV
│   ├── analyze_files.py            # Fase 2: ciclo OCR + analisi semantica
│   ├── migrate_to_minio.py         # Fase 3: migrazione e tagging su MinIO
│   ├── verify_integrity.py         # Verifica hash e consistenza archivio
│   └── utils.py                    # Funzioni comuni (log, OCR, API LLM)
│
├── 📂 config/                       # Configurazioni e modelli di prompt
│   ├── prompts/                    # Prompt per l’analisi semantica
│   │   ├── titolario.txt
│   │   └── few_shot_examples.json
│   ├── logging.yaml                # Configurazione del logging
│   ├── minio_config.yaml           # Parametri MinIO
│   └── ocr_config.yaml             # Configurazione OCR
│
├── 📂 data/                         # Dati locali temporanei
│   ├── raw/                        # Copia temporanea su NVMe
│   ├── output/                     # CSV e JSON di output
│   └── logs/                       # Log di esecuzione (non committare)
│
└── 📂 docs/                         # Documentazione tecnica
    ├── architecture.md             # Architettura e flusso logico
    ├── setup_guide.md              # Installazione e requisiti hardware
    ├── usage_examples.md           # Esempi di prompt e output
    └── retention_policy.md         # Regole di conservazione e scarto logico
 

## 🧩 Architettura in 3 Fasi

### **Fase 1 – Preparazione e Ingestione Dati**
- Copia dei 400 GB su SSD NVMe locale.  
- Creazione di un **indice CSV** con:
  - Percorso assoluto
  - Nome file
  - Estensione
  - Dimensione (byte)
- Inizializzazione delle colonne di output (`Stato_Analisi`, `Tag_S3_Rilevanza`, `Chiave_MinIO_Finale`).

> 🔧 Script: `scripts/ingest_data.py`

---

### **Fase 2 – Analisi Semantica e OCR**
Loop principale (≈200.000 iterazioni):

| Step | Funzione | Tecnologia |
|------|-----------|------------|
| A | Lettura CSV e filtro file da analizzare | Python / Pandas |
| B | OCR o parsing tecnico (PDF, DWG, SHP) | Tesseract GPU / PyMuPDF |
| C | Costruzione prompt con titolario e few-shot | Prompt template |
| D | Analisi semantica e classificazione | API LLM (Gemini/GPT-4o) |
| E | Scrittura risultato su CSV | Python |

Output per ogni file:
```json
{
  "TitoloArchivistico": "6.01.02 - Urbanistica",
  "ID_Progetto": "PZZA_MARCONI_2018",
  "Rilevanza": "ATTO_FINALE"
}
````

> 🔧 Script: `scripts/analyze_files.py`

---

### **Fase 3 – Migrazione e Policy in MinIO**

Dopo l’analisi completa:

* Upload in MinIO con struttura logica:

  ```
  /TitoloArchivistico/ID_Progetto/NomeFile
  ```
* Tagging S3:

  * `rilevanza=ATTO_FINALE|SCARTO_TECNICO`
  * `protocollo=<numero>`
  * `id_progetto=<stringa>`
* Applicazione policy WORM o Governance (retention 10 anni o scarto breve)
* Aggiornamento CSV finale con stato e hash file.

> 🔧 Script: `scripts/migrate_to_minio.py`

---

## ⚙️ Requisiti Tecnici

| Componente    | Versione Consigliata                 |
| ------------- | ------------------------------------ |
| Python        | ≥ 3.11                               |
| CUDA Toolkit  | ≥ 12.5                               |
| MinIO         | Ultima LTS                           |
| Tesseract OCR | Compilato con GPU                    |
| NVIDIA GPU    | RTX 5090 o superiore                 |
| NVMe          | ≥ 1 TB                               |
| LLM           | Gemini / GPT-4o (API key via `.env`) |

---

## 🔑 Variabili d’Ambiente

File `.env` (da creare a partire da `.env.example`):

```bash
# LLM API
LLM_PROVIDER=gpt4o
LLM_API_KEY=<chiave_api>

# MinIO
MINIO_ENDPOINT=https://minio.local
MINIO_ACCESS_KEY=<access_key>
MINIO_SECRET_KEY=<secret_key>
MINIO_BUCKET=archivio-mag

# Percorsi
DATA_PATH=/mnt/nvme/data
OUTPUT_CSV=data/output/analisi.csv
LOG_PATH=data/logs

# GPU
CUDA_VISIBLE_DEVICES=0
```

---

## 🧰 Installazione

```bash
git clone https://github.com/comune-magliano/Archivio-Semantico-AI.git
cd Archivio-Semantico-AI

python3 -m venv venv
source venv/bin/activate

pip install -r requirements.txt
cp .env.example .env
nano .env   # Inserisci chiavi e percorsi
```

---

## ▶️ Esecuzione del Workflow

```bash
# 1️⃣ Indicizzazione
python scripts/ingest_data.py

# 2️⃣ Analisi AI
python scripts/analyze_files.py

# 3️⃣ Migrazione in MinIO
python scripts/migrate_to_minio.py
```

---

## 🧾 Output Finale

* `data/output/analisi.csv`
  → Contiene per ogni file: titolo archivistico, stato, hash e percorso MinIO
* Archiviazione in **MinIO** con struttura e tagging coerente
* Log di esecuzione in `data/logs/`

---

## 📜 Licenza

Distribuito con licenza **MIT** — libero utilizzo e modifica a fini istituzionali.

---

## 👥 Autori

Progetto ideato e sviluppato per il
**Comune di Magliano in Toscana**
Settore Tecnico & Innovazione Digitale

---

> 💡 *“Un archivio intelligente non conserva solo documenti, ma memoria viva del territorio.”*

```

---

Vuoi che ti generi anche il file `.env.example` e il `requirements.txt` di base (con librerie per OCR, MinIO e LLM)?  
Posso scriverteli subito, così il progetto diventa **direttamente eseguibile**.
```
