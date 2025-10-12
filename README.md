🧠 Archivio-Semantico-AI: Classificazione Archivistica Basata su Metadati (LLM Locale)
Sistema AI per l'analisi semantica, classificazione e normalizzazione di archivi documentali. Il progetto sfrutta LLM (Large Language Models) ospitati localmente per interpretare e mappare il contesto d'archivio (percorsi cartelle, nomi file, campi CSV) allo schema del Titolario, assicurando la massima privacy e l'eliminazione dei servizi cloud esterni.
✨ Caratteristiche e Vantaggi Chiave
| Aspetto | Focus Privacy e Funzionalità |
|---|---|
| Classificazione Contestuale | L'LLM valuta la stringa del percorso del file (/nas/urbanistica/progetti/pratica_piazza_marconi/) e il nome file (o i campi CSV) per assegnare il codice archivistico più appropriato dal Titolario. |
| LLM 100% Locale | Tutta l'analisi dei metadati avviene on-premise tramite vLLM (o Ollama), utilizzando modelli come Llama o Mistral. Zero invio di dati a servizi cloud esterni. |
| Few-Shot Learning | L'LLM viene addestrato con circa 100 casi studio (esempi di percorsi/nomi vs. codice Titolario) per garantire l'accuratezza nella classificazione specifica del tuo archivio. |
| Normalizzazione Dati | Uniforma campi disomogenei come "Oggetto" (dai CSV) in formati standardizzati e concisi. |
| Migrazione S3 Integrata | Upload sicuro su MinIO con organizzazione automatica del percorso (s3://bucket/codice_titolario/progetto/) e applicazione di Policy di Retention (WORM). |
🏗️ Architettura del Workflow
L'architettura è modulare e supporta l'analisi di due tipi di input, entrambi basati sulla valutazione delle stringhe di metadati.
Flusso 1: Classificazione File (Metadati Archiviazione)
Questo flusso gestisce l'organizzazione dei file sul filesystem.
 * Ingestione: Scansione NAS/Filesystem e indicizzazione di Percorso Completo e Nome File in un CSV.
 * Analisi LLM: Lo script analyze_files.py invia Percorso + Nome File all'LLM Locale per ottenere la classificazione JSON.
 * Migrazione: Upload finale in MinIO, utilizzando il codice archivistico classificato per definire la destinazione.
Flusso 2: Analisi Record Gestionali (CSV)
Questo flusso gestisce la normalizzazione di una base dati esistente.
 * Input CSV: Legge il CSV grezzo (campi come Oggetto, Mittente, Data).
 * Analisi LLM: Lo script analyze_records_llm.py invia i campi del record all'LLM Locale che restituisce la classificazione normalizzata in JSON.
 * Output CSV: Salvataggio del CSV arricchito con i nuovi campi normalizzati (AI_titolo_archivistico, ecc.).
⚙️ Requisiti Tecnici
Hardware Raccomandato (Focus Inferenza LLM)
| Componente | Specifiche Ottimali | Note |
|---|---|---|
| GPU (per LLM) | NVIDIA RTX 4070 / 5090 (12GB+ VRAM) | Necessaria per ospitare modelli LLM di medie dimensioni. |
| CPU | Intel i9 / AMD Ryzen 9 | Per gestire l'elaborazione parallela. |
| Storage | 1 TB NVMe Gen4 | Per la cache dati e l'indicizzazione veloce. |
Software Essenziale
| Requisito | Versione | Note |
|---|---|---|
| Python | \geq 3.11 | Ambiente di esecuzione principale. |
| LLM Host | vLLM o Ollama | Per esporre il modello locale come API. |
| MinIO Server | LTS | Storage Object S3 compatibile. |
🛠️ Configurazione Few-Shot e Titolario
Il successo del modello dipende dai dati di addestramento forniti tramite il prompt.
1. Titolario Archivistico
Aggiorna il file config/prompts/titolario.txt con la tua struttura gerarchica completa.
2. Casi Studio (Few-Shot Examples)
Gli esempi addestrano l'LLM a interpretare i tuoi specifici pattern di denominazione.
 * Per Classificazione File: Crea coppie Percorso/Nome File \rightarrow JSON Classificazione target (salva in config/prompts/few_shot_files.json).
 * Per Analisi CSV: Crea coppie Campi CSV \rightarrow JSON Classificazione normalizzata (salva in config/prompts/few_shot_records.json).
▶️ Workflow Esecutivo
A. Classificazione di File Basata sul Percorso
# 1. Indicizzazione
python scripts/ingest_data.py --source /mnt/nas/archivio --dest /mnt/nvme/cache --output data/output/indice.csv

# 2. Analisi e Classificazione (LLM Locale)
python scripts/analyze_files.py --input data/output/indice.csv --workers 8

# 3. Migrazione Finale MinIO
python scripts/migrate_to_minio.py --input data/output/analisi.csv --apply-retention

B. Analisi di Record Gestionali (CSV)
# Analizza i record CSV (Oggetto, Mittente, etc.)
python scripts/analyze_records_llm.py \
  --input data/input/archivio_gestionale_raw.csv \
  --output data/output/archivio_gestionale_classificato.csv \
  --workers 8

🤝 Contribuire
Contributi per l'ottimizzazione dell'inferenza LLM locale (migliori configurazioni vLLM, supporto ad altri modelli) e l'adattamento a nuove strutture di metadati sono i benvenuti.
📜 Licenza
Distribuito con licenza MIT.
Made with ❤️ in Tuscany 🇮🇹
