# 🧠 Archivio-Semantico-AI  
### Classificazione Archivistica Basata su Metadati (LLM Locale)

Sistema AI per **analisi semantica, classificazione e normalizzazione** di archivi documentali.  
Il progetto utilizza **LLM locali (on-premise)** per interpretare e mappare i metadati (percorsi, nomi file, campi CSV) secondo lo **schema del Titolario**, garantendo **massima privacy** e nessun invio di dati verso servizi cloud esterni.

---

## ✨ Caratteristiche Principali

| Aspetto | Descrizione |
|---|---|
| **Classificazione Contestuale** | L’LLM analizza percorso e nome file (es. `/nas/urbanistica/progetti/pratica_piazza_marconi/`) per assegnare il codice archivistico corretto. |
| **LLM 100% Locale** | Tutta l’elaborazione avviene su server interni (vLLM o Ollama) con modelli come Llama o Mistral. Nessun cloud. |
| **Few-Shot Learning** | Addestramento con esempi reali del tuo archivio per massima accuratezza. |
| **Normalizzazione Dati** | Uniforma campi eterogenei dei CSV (es. “Oggetto”, “Mittente”) in formati coerenti. |
| **Migrazione S3 Integrata** | Organizza automaticamente i file in MinIO (`s3://bucket/codice_titolario/progetto/`) con policy WORM. |

---

## 🏗️ Architettura del Workflow

Il sistema gestisce due flussi basati su **analisi testuale dei metadati**:

### 🔹 Flusso 1: Classificazione File (Metadati Archiviazione)
1. **Ingestione:** scansione del filesystem (NAS) → CSV con percorso e nome file.  
2. **Analisi LLM:** `analyze_files.py` invia i dati all’LLM locale e riceve un JSON con classificazione archivistica.  
3. **Migrazione:** caricamento su MinIO organizzato per codice Titolario.

### 🔹 Flusso 2: Analisi Record Gestionali (CSV)
1. **Input CSV:** campi come “Oggetto”, “Mittente”, “Data”.  
2. **Analisi LLM:** `analyze_records_llm.py` restituisce la classificazione normalizzata in JSON.  
3. **Output CSV:** archivio arricchito con i nuovi campi AI.

---

## ⚙️ Requisiti Tecnici

### 💻 Hardware Raccomandato
| Componente | Specifiche Consigliate | Note |
|---|---|---|
| **GPU (per LLM)** | NVIDIA RTX 4070 / 5090 (12GB+ VRAM) | Per modelli di medie dimensioni. |
| **CPU** | Intel i9 / AMD Ryzen 9 | Elaborazioni parallele. |
| **Storage** | 1 TB NVMe Gen4 | Cache e indicizzazione. |

### 🧩 Software Essenziale
| Requisito | Versione | Descrizione |
|---|---|---|
| **Python** | ≥ 3.11 | Ambiente principale. |
| **LLM Host** | vLLM o Ollama | Esegue il modello locale come API. |
| **MinIO** | LTS | Storage S3 compatibile per archiviazione e retention. |

---

## 🧠 Configurazione Few-Shot e Titolario

### 1. **Titolario Archivistico**
Aggiorna:

config/prompts/titolario.txt

con la struttura gerarchica del tuo Titolario.

### 2. **Casi Studio (Few-Shot Examples)**
Fornisci esempi di addestramento per i tuoi pattern archivistici:

| Tipo | File | Descrizione |
|---|---|---|
| **File System** | `config/prompts/few_shot_files.json` | Coppie `Percorso/Nome File → JSON Classificazione` |
| **CSV Gestionale** | `config/prompts/few_shot_records.json` | Coppie `Campi CSV → JSON Normalizzato` |

---

## ▶️ Workflow Esecutivo

### A. Classificazione File
```bash
# 1. Indicizzazione
python scripts/ingest_data.py --source /mnt/nas/archivio --dest /mnt/nvme/cache --output data/output/indice.csv

# 2. Analisi LLM Locale
python scripts/analyze_files.py --input data/output/indice.csv --workers 8

# 3. Migrazione su MinIO
python scripts/migrate_to_minio.py --input data/output/analisi.csv --apply-retention

B. Analisi Record Gestionali (CSV)

python scripts/analyze_records_llm.py \
  --input data/input/archivio_gestionale_raw.csv \
  --output data/output/archivio_gestionale_classificato.csv \
  --workers 8


⸻

🐳 Deploy Locale con Docker

Il sistema è completamente contenuto in Docker e può essere eseguito in LAN senza accesso a Internet.

📦 docker-compose.yml

version: "3.9"

services:
  llm:
    image: ollama/ollama:latest
    container_name: llm_local
    restart: unless-stopped
    volumes:
      - ./models:/root/.ollama
    ports:
      - "11434:11434"
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
    environment:
      - OLLAMA_MODELS=/root/.ollama
    command: ["serve"]

  minio:
    image: minio/minio:latest
    container_name: minio
    restart: unless-stopped
    volumes:
      - ./minio_data:/data
    environment:
      - MINIO_ROOT_USER=admin
      - MINIO_ROOT_PASSWORD=admin123
    command: server /data --console-address ":9001"
    ports:
      - "9000:9000"
      - "9001:9001"

  app:
    build: ./app
    container_name: archivio_semantico
    restart: unless-stopped
    depends_on:
      - llm
      - minio
    volumes:
      - ./data:/app/data
      - ./config:/app/config
      - ./scripts:/app/scripts
    environment:
      - LLM_API=http://llm:11434/api/generate
      - MINIO_ENDPOINT=http://minio:9000
      - MINIO_USER=admin
      - MINIO_PASSWORD=admin123
    command: tail -f /dev/null

📁 Struttura Raccomandata del Progetto

Archivio-Semantico-AI/
├── app/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── scripts/
│       ├── ingest_data.py
│       ├── analyze_files.py
│       ├── analyze_records_llm.py
│       └── migrate_to_minio.py
├── config/
│   └── prompts/
│       ├── titolario.txt
│       ├── few_shot_files.json
│       └── few_shot_records.json
├── data/
│   ├── input/
│   ├── output/
│   └── cache/
├── docker-compose.yml
└── README.md

🐍 Esempio app/Dockerfile

FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY scripts/ ./scripts/
COPY config/ ./config/
COPY data/ ./data/

CMD ["bash"]

🧾 Esempio requirements.txt

requests
pandas
minio
tqdm

🚀 Avvio del Sistema

# 1. Avvia tutti i servizi
docker compose up -d

# 2. Accedi al container app
docker exec -it archivio_semantico bash

# 3. Lancia gli script di analisi
python scripts/analyze_files.py --input data/output/indice.csv --workers 8


⸻

🤝 Contributi

Contributi per:
	•	ottimizzare l’inferenza LLM locale (vLLM, Ollama, quantizzazione)
	•	aggiungere supporto a nuovi formati di metadati
	•	migliorare la gestione dei dataset e l’interfaccia n8n

sono benvenuti!
Apri una pull request o segnala un’issue.

⸻

📜 Licenza

Distribuito con licenza MIT.
Made with ❤️ in Tuscany 🇮🇹

---

Vuoi che aggiunga anche un **file `.env` di esempio** per gestire in modo sicuro le credenziali (LLM, MinIO, ecc.) nel Docker Compose? Sarebbe utile per deploy in ambienti pubblici o multiutente.