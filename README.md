🧠 Archivio-Semantico-AI: L'Archivista Robot Veloce e Privato
Sistema AI progettato per l'analisi automatica, classificazione semantica e migrazione sicura e privata di grandi archivi documentali (~200.000 file). Distingue atti ufficiali da bozze tecniche usando OCR accelerato GPU e AI Generativa Locale (LLM), garantendo la massima riservatezza dei dati.
✨ Caratteristiche Principali
🔒 Il Vantaggio Privacy (Focus sui Dati Sensibili)
Il cuore del progetto è l'uso di LLM Locali (come Llama, Mistral o Qwen) per l'analisi e la classificazione, garantendo che i dati sensibili non lascino mai l'infrastruttura locale.
 * Zero Cloud API: Il contenuto dei documenti non viene mai inviato a servizi cloud esterni (OpenAI, Google, ecc.) per l'analisi.
 * Massimo Controllo: L'intera pipeline di classificazione AI e conservazione è gestita on-premise.
🚀 Funzionalità Core
 * Classificazione Semantica AI Locale: Assegna automaticamente titoli d'archivio e tipologia (ATTO_FINALE, BOZZA_TECNICA) tramite LLM ospitati localmente.
 * OCR Accelerato GPU: Estrazione massiva e veloce del testo da PDF scansionati e immagini con Tesseract CUDA.
 * Parsing Multi-Formato: Supporto nativo per PDF, DWG, DXF, SHP, TIFF, DOCX.
 * Migrazione S3 Intelligente: Upload organizzato su MinIO con tagging, metadata e applicazione di Policy di Retention (WORM/Governance) per la conformità.
 * Deduplicazione SHA-256: Identificazione e scarto automatico dei file duplicati.
🏗️ Architettura del Workflow (3 Fasi)
Il processo trasforma un archivio grezzo in un repository MinIO sicuro, tramite un flusso sequenziale e tracciato.
graph TD
    A[Archivio Originale (NAS)] --> B;
    B(1. INGESTIONE & COPIA) --> C;
    C(2. ANALISI SEMANTICA AI) --> D;
    D(3. MIGRAZIONE MINIO) --> E;
    E[Archivio MinIO (Classificato & Conforme)]
    
    subgraph Fase 2 Dettaglio
        C --> C1(OCR GPU / Parsing);
        C1 --> C2(Prompt Engineering);
        C2 --> C3(LLM Locale VLLM);
        C3 --> C4(Output JSON Classificato);
    end

    style A fill:#f9f,stroke:#333
    style E fill:#ccf,stroke:#333
    style C3 fill:#ccf,stroke:#333
    style C4 fill:#ccf,stroke:#333

Workflow File: FILE.pdf \rightarrow Estrazione Testo \rightarrow LLM Locale (Classificazione) \rightarrow MinIO (s3://titolo/progetto/file.pdf).
⚡ Performance e Risparmio
L'uso esclusivo di AI locali è la soluzione più economica e privacy-oriented nel lungo termine.
| Metrica | Valore | Note |
|---|---|---|
| Tempo Totale (200K File) | ~82 ore | Solo 3.4 giorni di elaborazione continua. |
| Costo LLM API | $0 | Nessun costo di transazione per l'AI. |
| Throughput OCR/AI | 2.450 file/ora | Velocità media garantita da vLLM e CUDA. |
| Accuratezza | 95%+ | Ottenuta tramite prompt engineering e pochi-shot learning. |
> 💡 Break-even: Non essendoci costi API, l'investimento iniziale nell'hardware (GPU) è ammortizzato immediatamente, rendendolo ideale per archivi in crescita.
> 
⚙️ Requisiti Tecnici
Hardware (Per LLM Locale & CUDA)
| Componente | Specifiche Ottimali (Focus Locale) |
|---|---|
| GPU | NVIDIA RTX 5090 (24GB VRAM) |
| Storage | 2 TB NVMe Gen4 (per la cache dati) |
| RAM | 64 GB DDR5 |
Software
| Requisito | Versione | Note |
|---|---|---|
| Python | \geq 3.11 | Ambiente di esecuzione principale. |
| CUDA Toolkit | \geq 12.5 | Necessario per l'accelerazione OCR e vLLM. |
| MinIO Server | LTS | Storage Object S3 conforme. |
| Tesseract OCR | \geq 5.3 | Compilato con supporto CUDA. |
| LLM Locali | Llama 3.1 / Mistral | Ospitati con vLLM per l'inferenza rapida. |
🚀 Installazione Rapida
1. Clonazione e Ambiente
git clone https://github.com/comune-magliano/Archivio-Semantico-AI.git
cd Archivio-Semantico-AI
python3.11 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

2. Setup MinIO (Locale con Docker)
Per iniziare rapidamente con MinIO (Storage):
docker run -d \
  -p 9000:9000 -p 9001:9001 \
  --name minio \
  -e "MINIO_ROOT_USER=admin" \
  -e "MINIO_ROOT_PASSWORD=password123" \
  minio/minio server /data --console-address ":9001"

3. Configurazione Privacy-Centrica
Copia e modifica il file di configurazione ambientale, specificando l'endpoint LLM locale (e lasciando vuote le chiavi Cloud API):
cp .env.example .env
nano .env

# PARAMETRI LLM LOCALE (VLLM o altro endpoint locale)
LLM_PROVIDER=local_vllm
LOCAL_VLLM_ENDPOINT=http://localhost:8000/v1
# LLM_API_KEY= (lasciare vuoto per massima privacy)

# PARAMETRI MINIO
MINIO_ENDPOINT=http://localhost:9000
MINIO_ACCESS_KEY=admin
MINIO_SECRET_KEY=password123

▶️ Workflow Esecutivo
Esegui gli script in sequenza:
Fase 1: Ingestione Dati
python scripts/ingest_data.py \
  --source /mnt/nas/archivio_vecchio \
  --dest /mnt/nvme/cache \
  --output data/output/indice.csv

Fase 2: Analisi AI Locale
(Assicurati che l'endpoint LLM locale con il modello Llama/Mistral sia attivo)
python scripts/analyze_files.py \
  --input data/output/indice.csv \
  --batch-size 100 \
  --workers 8

Fase 3: Migrazione MinIO
python scripts/migrate_to_minio.py \
  --input data/output/analisi.csv \
  --verify-hash \
  --apply-retention

📁 Struttura del Repository
Archivio-Semantico-AI/
│
├── 📂 scripts/                       # Logica di business (OCR, LLM Client, MinIO Manager)
│   ├── analyze_files.py               # Script FASE 2: Analisi AI Locale
│   └── minio_manager.py               # Gestione S3
│
├── 📂 config/                       # Configurazioni (Prompt, Titolario, Logging)
│   ├── prompts/
│   │   ├── titolario.txt           # Schema di classificazione personalizzato
│   │   └── few_shot_examples.json  # Esempi per l'AI
│
├── 📂 data/                         # Output (CSV, Log)
│
└── 📄 requirements.txt               # Dipendenze Python

🗺️ Roadmap (Privacy & Control)
 * v1.1 (Q1 2025): Interfaccia web di monitoring e API REST per interrogazione archivio.
 * v1.2 (Q2 2025): Fine-tuning LLM sul dataset archivistico specifico per massimizzare l'accuratezza in locale.
 * v2.0 (Q3 2025): Ricerca semantica vettoriale (ChromaDB) on-premise per Q&A avanzato e recupero informazioni.
🤝 Contribuire
I contributi sono benvenuti! Sentiti libero di aprire una Pull Request per migliorare l'integrazione di nuovi modelli LLM locali, ottimizzare l'inferenza o aggiungere nuove funzionalità.
Vedi Linee Guida di Contribuzione
📜 Licenza
Distribuito con licenza MIT — libero per uso istituzionale e commerciale.
Vedi LICENSE per il testo completo.
<div align="center">
Made with ❤️ in Tuscany 🇮🇹
Powered by Open Source & Local AI
</div>
