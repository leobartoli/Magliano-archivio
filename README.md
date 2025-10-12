Perfetto ✅
Ecco il file README.md completo, già formattato per l’inclusione diretta nel repository GitHub del progetto Archivio-Semantico-AI.
È pulito, coerente e pronto per l’uso pubblico — senza dati sensibili ma con tutte le istruzioni operative e la struttura chiara.

⸻


# 🧠 Archivio-Semantico-AI

Sistema intelligente per l’analisi, classificazione e migrazione automatica di grandi archivi documentali (≈200.000 file) verso un archivio **MinIO conforme**, distinguendo **atti ufficiali** da **bozze tecniche** tramite **OCR** e **AI semantica**.

---

## 🚀 Obiettivo del Progetto

Il progetto **Archivio-Semantico-AI** automatizza l’intero ciclo di vita di un archivio tecnico o amministrativo, con un approccio basato su:

- **Analisi ibrida OCR + AI semantica**
- **GPU NVIDIA** (es. RTX 5090) per elaborazioni accelerate
- **SSD NVMe** per I/O ad alte prestazioni
- **LLM (Gemini / GPT-4o)** per classificazione e tagging semantico
- **MinIO** come archivio finale conforme S3 con policy di retention

---

## 📁 Struttura del Repository

Archivio-Semantico-AI/
│
├── 📄 README.md                     # Documentazione principale
├── 📄 .env.example                  # Template variabili d’ambiente
├── 📄 requirements.txt              # Dipendenze Python
├── 📄 LICENSE                       # Licenza MIT
├── 📄 .gitignore                    # Esclusioni standard
│
├── 📂 scripts/
│   ├── ingest_data.py              # Fase 1: indicizzazione
│   ├── analyze_files.py            # Fase 2: OCR + AI
│   ├── migrate_to_minio.py         # Fase 3: migrazione su MinIO
│   ├── verify_integrity.py         # Verifica hash e consistenza
│   └── utils.py                    # Funzioni comuni
│
├── 📂 config/
│   ├── prompts/
│   │   ├── titolario.txt
│   │   └── few_shot_examples.json
│   ├── logging.yaml
│   ├── minio_config.yaml
│   └── ocr_config.yaml
│
├── 📂 data/
│   ├── raw/                        # Copia temporanea su NVMe
│   ├── output/                     # CSV e JSON di output
│   └── logs/                       # Log di esecuzione
│
└── 📂 docs/
├── architecture.md
├── setup_guide.md
├── usage_examples.md
└── retention_policy.md

---

## 🧩 Architettura in 3 Fasi

### **Fase 1 — Ingestione**
- Copia dei dati su SSD NVMe locale  
- Creazione indice CSV con:
  - Percorso assoluto  
  - Nome file  
  - Estensione e dimensione  
  - Colonne di stato e output  
> 🔧 Script: `scripts/ingest_data.py`

---

### **Fase 2 — Analisi Semantica e OCR**
Loop principale (≈200 000 iterazioni):

| Step | Funzione | Tecnologia |
|------|-----------|------------|
| A | Lettura indice CSV | Pandas |
| B | OCR o parsing PDF/DWG/SHP | Tesseract GPU / PyMuPDF |
| C | Costruzione prompt con titolario | Template |
| D | Analisi semantica | API LLM (Gemini / GPT-4o) |
| E | Scrittura risultati | CSV |

Esempio output:
```json
{
  "TitoloArchivistico": "6.01.02 - Urbanistica",
  "ID_Progetto": "PZZA_MARCONI_2018",
  "Rilevanza": "ATTO_FINALE"
}

🔧 Script: scripts/analyze_files.py

⸻

Fase 3 — Migrazione su MinIO
	•	Upload organizzato in /TitoloArchivistico/ID_Progetto/NomeFile
	•	Tagging S3 automatico:
	•	rilevanza=ATTO_FINALE|SCARTO_TECNICO
	•	protocollo=<numero>
	•	id_progetto=<stringa>
	•	Applicazione policy WORM/Governance
	•	Aggiornamento CSV finale con hash e stato

🔧 Script: scripts/migrate_to_minio.py

⸻

⚙️ Requisiti Tecnici

Componente	Versione Consigliata
Python	≥ 3.11
CUDA Toolkit	≥ 12.5
MinIO	LTS
Tesseract OCR	con supporto GPU
GPU NVIDIA	RTX 5090 o superiore
NVMe	≥ 1 TB
LLM	Gemini / GPT-4o


⸻

🔑 Variabili d’Ambiente

File .env (da creare da .env.example):

# LLM API
LLM_PROVIDER=gpt4o
LLM_API_KEY=<chiave_api>

# MinIO
MINIO_ENDPOINT=https://minio.local
MINIO_ACCESS_KEY=<access_key>
MINIO_SECRET_KEY=<secret_key>
MINIO_BUCKET=archivio-mag

# Percorsi locali
DATA_PATH=/mnt/nvme/data
OUTPUT_CSV=data/output/analisi.csv
LOG_PATH=data/logs

# GPU
CUDA_VISIBLE_DEVICES=0


⸻

🧰 Installazione

git clone https://github.com/comune-magliano/Archivio-Semantico-AI.git
cd Archivio-Semantico-AI

python3 -m venv venv
source venv/bin/activate

pip install -r requirements.txt

cp .env.example .env
nano .env  # Inserisci chiavi e percorsi


⸻

▶️ Esecuzione del Workflow

# 1️⃣ Indicizzazione
python scripts/ingest_data.py

# 2️⃣ Analisi AI
python scripts/analyze_files.py

# 3️⃣ Migrazione in MinIO
python scripts/migrate_to_minio.py


⸻

🧾 Output Finale
	•	data/output/analisi.csv
→ Titolo archivistico, stato, hash e percorso MinIO
	•	Archiviazione in MinIO con tagging coerente
	•	Log in data/logs/

⸻

📜 Licenza

Distribuito con licenza MIT — uso e modifica liberi per finalità istituzionali.

⸻

👥 Autori

Progetto ideato e sviluppato per il
Comune di Magliano in Toscana
Settore Tecnico & Innovazione Digitale

⸻

💡 “Un archivio intelligente non conserva solo documenti, ma memoria viva del territorio.”

---

Vuoi che ti generi anche i file complementari (`.env.example`, `requirements.txt` e `.gitignore`) in formato pronto per commit GitHub?  
Posso crearli subito con librerie OCR, MinIO e LLM già elencate.