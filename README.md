Certamente! Ecco il testo completo della soluzione in formato README.md, pronto per essere utilizzato in un repository GitHub, come richiesto.
📁 MinIO Archival Initializer - Titolario Comunale
Questo progetto fornisce uno script Python per automatizzare la creazione dell'intera struttura gerarchica (Titoli, Classi, Sottoclassi) di un Titolario di Classificazione Comunale all'interno di un'istanza MinIO.
Il codice si basa sull'analisi dettagliata del documento "Copia di definitivo titolario classificazione con fascicoli 2021.pdf", replicandone l'esatta struttura archivistica (fino al 4° livello di classificazione) e includendo la logica per le serie speciali (come i Fascicoli del Personale).
💡 Strategia di Esecuzione (Docker Compose)
Il problema principale è accedere all'istanza MinIO, che è isolata nel network Docker (backend).
La soluzione più pulita e professionale, che non richiede la modifica del tuo docker-compose.yml, è avviare un container Python temporaneo e iniettarlo nella rete Docker esistente. Questo container eseguirà lo script e si auto-distruggerà al termine.
| Parametro | Valore da Usare nel Container | Descrizione |
|---|---|---|
| Endpoint | minio:9000 | Nome del servizio Docker (minio) e porta interna. |
| Network | [Il tuo prefisso]_backend | Rete Docker necessaria per la comunicazione interna. |
1. Prerequisites
Assicurati di avere:
 * Docker e Docker Compose installati e funzionanti.
 * L'istanza MinIO avviata (docker compose up -d minio).
 * Le credenziali di root di MinIO (MINIO_ROOT_USER e MINIO_ROOT_PASSWORD).
2. Lo Script Python (create_titolario_minio.py)
Salva il seguente codice completo in un file chiamato create_titolario_minio.py nella stessa directory del tuo file docker-compose.yml.
#!/usr/bin/env python3
"""
Script per creare la struttura completa del Titolario Comunale su MinIO.
Crea bucket e cartelle seguendo l'esatta classificazione (Titolo, Classe, Sottoclasse)
del documento PDF fornito.
"""

from minio import Minio
from minio.error import S3Error
import os
from datetime import datetime

# ==============================================================================
# 1. Configurazione MinIO (Recuperata da variabili d'ambiente Docker)
# ==============================================================================

# L'endpoint corretto all'interno della rete Docker è 'minio:9000'
MINIO_ENDPOINT = os.getenv('MINIO_ENDPOINT', 'localhost:9000')
MINIO_ACCESS_KEY = os.getenv('MINIO_ACCESS_KEY', 'minioadmin')
MINIO_SECRET_KEY = os.getenv('MINIO_SECRET_KEY', 'minioadmin')
MINIO_SECURE = os.getenv('MINIO_SECURE', 'false').lower() == 'true'

# ==============================================================================
# 2. Struttura del Titolario (Estratta dal PDF)
# ==============================================================================

# Struttura: {Titolo: {Classe: {Sottoclasse: [Sub-Sottoclassi]}}}
TITOLARIO = {
    # TITOLO I - AMMINISTRAZIONE GENERALE
    "titolo-01-amministrazione-generale": {
        "1.1-legislazione-e-circolari-esplicative": {
            "1.1.1-normativa-varie": [],
            "1.1.2-decreti-emanati-dal-sindaco": [],
            "1.1.3-ordinanze-emanate-dal-sindaco": [],
            "1.1.4-circolari-varie": [],
            "1.1.5-pareri-richiesti-su-leggi-specifiche": []
        },
        "1.2-denominazione-territorio-e-confini-toponomastica": {
            "1.2.1-denominazione-del-comune": [],
            "1.2.2-confini-del-comune": [],
            "1.2.3-verbali-e-deliberazioni-commissione-toponomastica": [
                "1.2.3.1-intitolazione-nuove-vie",
                "1.2.3.2-richiesta-assegnazione-numero-civico"
            ],
            "1.2.4-attribuzione-del-titolo-di-citta": []
        },
        "1.3-statuto": {
            "1.3.1-statuto-comunale-redazione-modifiche-e-interpretazioni": []
        },
        "1.4-regolamenti": {
            "1.4.1-regolamento-ordinamento-uffici-e-servizi": [
                "1.4.1.1-organigramma-del-comune",
                "1.4.1.2-funzionigramma-del-comune"
            ],
            "1.4.2-regolamento-del-servizio-entrate": [
                "1.4.2.1-imposta-municipale-propria-imu",
                "1.4.2.2-tassa-sui-rifiuti-tari",
                "1.4.2.3-imposta-comunale-sulla-pubblicita-cosap",
                "1.4.2.4-addizionale-comunale-irpef",
                "1.4.2.5-diritti-sulle-affissioni",
                "1.4.2.6-imposta-di-soggiorno"
            ],
            "1.4.3-regolamenti-del-servizio-cultura": [
                "1.4.3.1-concessione-contributi-associazioni"
            ],
            "1.4.4-regolamenti-dei-servizi-educativi": [
                "1.4.4.1-regolamento-autorizzazione-funzionamento-prima-infanzia",
                "1.4.4.2-regolamento-nidi",
                "1.4.4.3-regolamento-commissione-mensa"
            ],
            "1.4.5-regolamento-dei-servizi-demografici": [
                "1.4.5.1-regolamento-comunale-matrimonio-civile",
                "1.4.5.2-regolamento-di-polizia-mortuaria"
            ],
            "1.4.6-regolamento-dei-servizi-sociali": [
                "1.4.6.1-regolamento-accesso-prestazioni-agevolate",
                "1.4.6.2-regolamento-poverta",
                "1.4.6.3-regolamento-orti-sociali",
                "1.4.6.4-regolamento-integrazione-rette",
                "1.4.6.5-regolamento-duso-alloggi-erp",
                "1.4.6.6-regolamento-di-taxi-sociale",
                "1.4.6.7-regolamento-organizzazione-sportello-sociale",
                "1.4.6.8-regolamento-servizio-assistenza-domiciliare",
                "1.4.6.9-regolamento-consulta-del-volontariato",
                "1.4.6.10-regolamento-concessione-di-sovvenzioni-ad-associazioni",
                "1.4.6.12-regolamento-gestione-centro-diurno-anziani",
                "1.4.6.13-regolamento-comitato-di-distretto",
                "1.4.6.14-regolamento-centro-per-le-famiglie",
                "1.4.6.15-regolamento-centro-diurno-assistenziale",
                "1.4.6.16-regolamento-casa-famiglia",
                "1.4.6.17-regolamento-assegni-a-sostegno-della-domiciliarita",
                "1.4.6.18-regolamento-assegnazioni-alloggi-erp"
            ],
            "1.4.7-regolamento-del-servizio-ambiente": [
                "1.4.7.1-registro-benessere-animale",
                "1.4.7.2-registro-servizio-gestione-rifiuti-urbani"
            ],
            "1.4.8-regolamenti-per-il-funzionamento-del-consiglio-comunale": [
                "1.4.8.1-commissione-n-1",
                "1.4.8.2-commissione-n-2",
                "1.4.8.3-commissione-n-3"
            ],
            "1.4.9-regolamenti-generali": [
                "1.4.9.1-regolamento-sul-trattamento-dei-dati-personali",
                "1.4.9.2-nuovo-regolamento-contratti",
                "1.4.9.3-regolamento-accesso-documentale",
                "1.4.9.4-regolamento-alienazioni"
            ],
            "1.4.10-regolamenti-del-personale": []
        },
        "1.5-stemma-gonfalone-sigillo": {
            "1.5.1-stemma-concessione-definizione-riconoscimento": [],
            "1.5.2-gonfalone-concessione-definizione-riconoscimento": [],
            "1.5.3-sigillo-concessione-definizione-riconoscimento": []
        },
        "1.6-archivio-generale": {
            "1.6.1-registro-di-protocollo-informatico": [],
            "1.6.2-registro-dell-albo-pretorio": [],
            "1.6.3-registro-delle-notifiche": [],
            "1.6.4-registro-di-notifica-presso-la-casa-comunale": [],
            "1.6.5-registro-di-accesso-atti-per-fini-amministrativi": [],
            "1.6.6-repertorio-dei-fascicoli": [],
            "1.6.7-organizzazione-del-servizio-ordinario-manuale-conservazione-etc": [],
            "1.6.8-interventi-straordinari-traslochi-restauri-gestione-servizi": [],
            "1.6.9-richieste-di-accesso-per-fini-amministrativi": [],
            "1.6.10-richieste-di-informazioni-archivistiche-e-studio": [],
            "1.6.11-richieste-di-pubblicazione-all-albo-pretorio": [],
            "1.6.12-registro-delle-spedizioni-e-delle-spese-postali": [],
            "1.6.13-ordinanze-emanate-dal-sindaco-repertorio": [],
            "1.6.14-decreti-del-sindaco-repertorio": [],
            "1.6.15-ordinanze-emanate-dai-dirigenti-repertorio": [],
            "1.6.16-determinazione-dei-dirigenti-repertorio": [],
            "1.6.17-deliberazione-del-consiglio-comunale-repertorio": [],
            "1.6.18-deliberazione-della-giunta-comunale-repertorio": [],
            "1.6.19-verbali-degli-altri-organi-collegiali-repertorio": [],
            "1.6.20-contratti-e-convenzioni-repertorio": [],
            "1.6.21-atti-rogati-dal-segretario-repertorio": []
        },
        "1.7-sistema-informativo": {
            "1.7.1-sistema-informativo-organizzazione": [],
            "1.7.2-statistiche-varie": []
        },
        "1.8-informazioni-e-relazioni-con-il-pubblico": {
            "1.8.1-iniziative-specifiche-urp-informagiovane": [],
            "1.8.2-reclami-dei-cittadini": [],
            "1.8.3-bandi-e-avvisi-di-stampa": [],
            "1.8.4-materiale-preparatorio-per-il-sito-web": []
        },
        "1.9-politica-del-personale-ordinamenti-uffici-e-servizi": {
            "1.9.1-attribuzioni-di-competenze-agli-uffici": [],
            "1.9.2-organigramma-del-comune": [],
            "1.9.3-funzionigramma-del-comune": [],
            "1.9.4-organizzazione-degli-uffici-orari-di-apertura-uffici-comunali": [],
            "1.9.5-orari-di-apertura-degli-uffici-e-attivita-esistenti-sul-territorio-comunale": [],
            "1.9.6-materiale-preparatorio-per-le-deliberazioni-materia-di-politica-del-personale": []
        },
        "1.10-relazioni-con-le-organizzazioni-sindacali": {
            "1.10.1-verbali-della-delegazione-trattante-per-la-contrattazione-integrativa": [],
            "1.10.2-rapporti-di-carattere-generale": [],
            "1.10.3-costituzione-delle-rappresentanze-del-personale": []
        },
        "1.11-controlli-esterni-interni": {
            "1.11.1-controlli-specifici": []
        },
        "1.12-editoria-e-attivita-informativo-promozionale": {
            "1.12.1-pubblicazioni-istituzionali-del-comune-raccolta-bibliografica": [],
            "1.12.2-comunicati-stampa": []
        },
        "1.13-cerimoniale-attivita-di-rappresentanza-onorificenze": {
            "1.13.1-iniziative-specifiche": [],
            "1.13.2-onorificenze-concesse-e-ricevute": [],
            "1.13.3-concessione-dell-uso-del-sigillo": []
        },
        "1.14-interventi-di-carattere-politico-e-umanitario-rapporti-istituzionali": {
            "1.14.1-iniziative-specifiche-gemellaggi-adesione-movimenti-opinione": [],
            "1.14.2-gemellaggi": [],
            "1.14.3-promozione-di-comitati": []
        },
        "1.15-forme-associative-partecipative-e-adesioni": {
            "1.15.1-costituzione-di-enti-controllati-dal-comune": [],
            "1.15.2-partecipazione-del-comune-a-enti-e-associazioni": []
        },
        "1.16-aree-e-citta-metropolitane": {
            "1.16.1-costituzione-e-rapporti-istituzionali": []
        },
        "1.17-associazionismo-e-partecipazione": {
            "1.17.1-associazioni-che-richiedono-iscrizione-all-albo": [],
            "1.17.2-albo-delle-associazioni": []
        }
    },
    
    # TITOLO II - ORGANI DI GOVERNO, GESTIONE, CONTROLLO, CONSULENZA E GARANZIA
    "titolo-02-organi-governo-gestione-controllo-consulenza-e-garanzia": {
        "2.1-sindaco": {
            "2.1.1-fascicolo-personale-durata-mandato": []
        },
        "2.2-vice-sindaco": {
            "2.2.1-fascicolo-personale-durata-mandato": []
        },
        "2.3-consiglio": {
            "2.3.1-fascicolo-personale-durata-mandato": [],
            "2.3.2-convocazione-del-consiglio-comunale-e-odg": [],
            "2.3.3-interrogazioni-e-mozioni-consiliari": [],
            "2.3.4-situazione-patrimoniale-titolari-di-cariche-elettive": [],
            "2.3.5-accesso-atti-consiglieri-comunali": []
        },
        "2.4-presidente-del-consiglio": {
            "2.4.1-fascicolo-personale-durata-mandato": []
        },
        "2.5-conferenza-dei-capigruppo-e-commissioni-del-consiglio": {
            "2.5.1-verbali-della-conferenza-dei-capigruppo": [],
            "2.5.2-verbale-delle-commissioni": []
        },
        "2.6-gruppi-consiliari": {
            "2.6.1-accreditamento-presso-il-consiglio": []
        },
        "2.7-giunta": {
            "2.7.1-nomine-revoche-e-dimissioni-degli-assessori": [],
            "2.7.2-convocazione-della-giunta-e-odg": []
        },
        "2.8-commissario-prefettizio-e-straordinario": {
            "2.8.1-fascicolo-personale": []
        },
        "2.9-segretario-e-vice-segretario": {
            "2.9.1-fascicolo-personale-durata-incarico": []
        },
        "2.10-direttore-generale-e-dirigenza": {
            "2.10.1-fascicolo-personale-durata-incarico": []
        },
        "2.11-revisori-dei-conti": {
            "2.11.1-fascicolo-personale-durata-incarico": [],
            "2.11.2-relazioni-repertorio-annuale": []
        },
        "2.12-difensore-civico": {
            "2.12.1-fascicolo-personale-durata-incarico": []
        },
        "2.13-commissario-ad-acta": {
            "2.13.1-fascicolo-personale-durata-incarico": []
        },
        "2.14-organi-di-controllo-interni": {
            "2.14.1-un-fascicolo-per-ogni-organo": []
        },
        "2.15-organi-consultivi": {
            "2.15.1-fascicoli-relativi-al-funzionamento": [],
            "2.15.2-relazione-degli-organi-consuntivi": []
        },
        "2.16-consigli-circoscrizionali": {},
        "2.17-presidente-dei-consigli-circoscrizionali": {},
        "2.18-organi-esecutivi-circoscrizionali": {},
        "2.19-commissioni-dei-consigli-circoscrizionali": {},
        "2.20-segretari-delle-circoscrizioni": {},
        "2.21-commissario-ad-acta-delle-circoscrizioni": {},
        "2.22-conferenza-del-presidente-di-quartiere": {}
    },
    
    # TITOLO III - RISORSE UMANE
    "titolo-03-risorse-umane": {
        "3.1-concorsi-selezione-colloqui": {
            "3.1.1-criteri-generali-e-normativa-per-il-reclutamento": [],
            "3.1.2-procedimenti-per-il-reclutamento-del-personale": [],
            "3.1.3-curriculum-vitae-inviati-per-richieste-di-assunzione": [],
            "3.1.4-domande-di-assunzione-pervenute-senza-indizione-di-concorso": []
        },
        "3.2-assunzioni-e-cessazioni": {
            "3.2.1-criteri-generali-e-normativa-per-le-assunzioni-e-cessazioni": [],
            "3.2.2-determinazioni-di-assunzioni-e-cessazione-dei-singoli": []
        },
        "3.3-comandi-distacchi-mobilita": {
            "3.3.1-criteri-generali-e-normativa-per-comandi-distacchi-e-mobilita": [],
            "3.3.2-determinazioni-di-comandi-distacchi-mobilita-inserite-nei-fascicoli-personali": []
        },
        "3.4-attribuzione-di-funzioni-ordini-di-servizio-e-missioni": {
            "3.4.1-criteri-generali-e-normativa-per-le-attribuzioni-di-funzioni": [],
            "3.4.2-determinazioni-di-attribuzione-di-funzioni-inserite-nei-fascicoli-personali": [],
            "3.4.3-determinazioni-di-missioni-inserite-nei-fascicoli-personali": [],
            "3.4.4-determinazione-di-ordini-di-servizio-inserite-nei-fascicoli-personali": [],
            "3.4.5-determinazione-di-autorizzazione-allo-svolgimento-di-incarichi-esterni": []
        },
        "3.5-inquadramenti-applicazione-dei-ccnl": {
            "3.5.1-criteri-generali-e-normativa-per-gli-inquadramenti": [],
            "3.5.2-determinazione-dei-ruoli-e-contratti-collettivi": [],
            "3.5.3-determinazioni-relative-ai-singoli-inserire-nei-rispettivi-fascicoli-personali": []
        },
        "3.6-retribuzioni-e-compensi": {
            "3.6.1-criteri-generali-e-normativi-per-le-retribuzioni-e-compensi": [],
            "3.6.2-anagrafe-delle-prestazioni-base-di-dati": [],
            "3.6.3-determinazioni-inserite-nei-singoli-fascicoli-personali": [],
            "3.6.4-ruoli-degli-stipendi": [],
            "3.6.5-provvedimenti-giudiziari-su-requisizione-dello-stipendio": [],
            "3.6.6-indennita-di-funzione-degli-amministratori": [],
            "3.6.7-rimborso-oneri-per-permessi-amministratori": [],
            "3.6.8-missioni-amministratori": [],
            "3.6.9-gettoni-di-presenza": [],
            "3.6.10-valutazione-dirigenti": [],
            "3.6.11-pagamento-compensi-incentivanti-ufficio-legale": [],
            "3.6.12-determinazioni-relativi-ai-singoli-per-la-definizione-delle-voci-accessorie": []
        },
        "3.7-trattamento-fiscale-contributivo-e-assicurativo": {
            "3.7.1-criteri-generali-e-normativi-per-gli-adempimenti-fiscali-contributivi": [],
            "3.7.2-trattamento-assicurativo-inserito-nei-singoli-fascicoli-personali": [],
            "3.7.3-trattamento-contributivo-inserito-nei-singoli-fascicoli-personali": [],
            "3.7.4-trattamento-fiscale-inserito-nei-singoli-fascicoli-personali": [],
            "3.7.5-assicurazione-obbligatoria-inserita-nei-singoli-fascicoli-personali": []
        },
        "3.8-tutela-della-salute-e-sicurezza-sul-luogo-di-lavoro": {
            "3.8.1-criteri-generali-e-normativa-per-la-tutela-della-salute-e-sicurezza": [],
            "3.8.2-rilevazione-dei-rischi": [],
            "3.8.3-prevenzione-infortuni": [],
            "3.8.4-registro-infortuni": [],
            "3.8.5-verbali-delle-rappresentanze-dei-lavoratori-per-la-sicurezza": [],
            "3.8.6-denuncia-di-infortunio-e-pratica-relativa-inserita-nei-fascicoli-personali": [],
            "3.8.7-fascicoli-relativi-alle-visite-mediche-ordinarie": []
        },
        "3.9-dichiarazione-di-infermita-ed-equo-indennizzo": {
            "3.9.1-criteri-generali-e-normative-per-la-dichiarazione-di-infermita": [],
            "3.9.2-dichiarazione-di-infermita-e-calcolo-dell-indennizzo-inserite-nel-fascicolo-personale": []
        },
        "3.10-indennita-premio-di-servizio-tfr-quiescenza": {
            "3.10.1-criteri-generali-e-normativa-per-il-trattamento-di-fine-rapporto": [],
            "3.10.2-trattamento-pensionistico-e-di-fine-rapporto-inserito-nel-fascicolo-personale": []
        },
        "3.11-servizi-al-personale-su-richiesta": {
            "3.11.1-criteri-generali-e-normativa-per-i-servizi-su-richiesta": [],
            "3.11.2-domande-di-servizi-su-richiesta-mensa-asilo-nido-centri-estivi": []
        },
        "3.12-orario-di-lavoro-presenza-e-assenze": {
            "3.12.1-criteri-generali-e-normative-per-le-assenze": [],
            "3.12.2-domande-e-dichiarazioni-sull-orario-inserito-nel-fascicolo-personale": [],
            "3.12.3-150-ore-diritto-allo-studio-inserito-nel-fascicolo-personale": [],
            "3.12.4-permessi-d-uscita-per-motivi-personali-inserito-nel-fascicolo-personale": [],
            "3.12.5-permessi-per-allattamento-inserito-nel-fascicolo-personale": [],
            "3.12.6-permessi-per-donazione-sangue-inserito-nel-fascicolo-personale": [],
            "3.12.7-permessi-per-motivi-sindacali-inserito-nel-fascicolo-personale": [],
            "3.12.8-opzioni-per-orario-particolare-part-time-inserito-nel-fascicolo-personale": [],
            "3.12.9-congedo-ordinario-inserito-nel-fascicolo-personale": [],
            "3.12.10-congedo-straordinario-per-motivi-di-salute-inserito-nel-fascicolo-personale": [],
            "3.12.11-congedo-straordinario-per-motivi-personali-e-familiari-inserito-nel-fascicolo-personale": [],
            "3.12.12-aspettativa-per-infermita-inserito-nel-fascicolo-personale": [],
            "3.12.13-aspettativa-per-mandato-parlamentare-o-altre-cariche-elettive-inserito-nel-fascicolo-personale": [],
            "3.12.14-aspettativa-obbligatoria-per-maternita-e-puerperio-inserito-nel-fascicolo-personale": [],
            "3.12.15-aspettativa-facoltativa-per-motivi-di-famiglia-inserito-nel-fascicolo-personale": [],
            "3.12.16-aspettativa-sindacale-inserito-nel-fascicolo-personale": [],
            "3.12.17-certificati-medici-inserito-nel-fascicolo-personale": [],
            "3.12.18-referti-delle-visite-di-controllo-inseriti-nel-fascicolo-personale": [],
            "3.12.19-fogli-firma-marcatempo-tabulati-elettronici-di-rilevazione-presenze": [],
            "3.12.20-rilevazioni-dell-assenza-per-sciopero": []
        },
        "3.13-giudizi-responsabilita-e-provvedimenti-disciplinari": {
            "3.13.1-criteri-generali-e-normative-per-i-provvedimenti-disciplinari": [],
            "3.13.2-provvedimenti-disciplinari-inseriti-nel-singolo-fascicolo-personale": []
        },
        "3.14-formazione-aggiornamento-professionale": {
            "3.14.1-criteri-generali-e-normativi-per-la-formazione-e-l-aggiornamento": [],
            "3.14.2-organizzazione-di-corsi-di-formazione-e-aggiornamento": [],
            "3.14.3-domande-invio-dei-dipendenti-a-corsi-inseriti-nel-singolo-fascicolo-personale": []
        },
        "3.15-collaboratori-esterni": {
            "3.15.1-criteri-generali-e-normative-per-il-trattamento-dei-collaboratori-esterni": [],
            "3.15.2-elenco-degli-incarichi-conferiti": []
        }
    },
    
    # TITOLO IV - RISORSE FINANZIARIE E PATRIMONIALI
    "titolo-04-risorse-finanziarie-e-patrimoniali": {
        "4.1-bilancio-preventivo-e-peg": {
            "4.1.1-bilancio-preventivo-e-allegati": [],
            "4.1.2-peg-piano-esecutivo-di-gestione": [],
            "4.1.3-carteggio-uffici-per-formazione-bilancio-e-peg": []
        },
        "4.2-gestione-del-bilancio-e-del-peg-con-variazioni": {
            "4.2.1-gestione-del-bilancio-un-fascicolo-per-ciascuna-variazione": []
        },
        "4.3-gestione-delle-entrate-accertamento-riscossione-versamento": {
            "4.3.1-fascicoli-personali-dei-contribuenti-comunali": [],
            "4.3.2-ruolo-ici": [],
            "4.3.3-ruolo-imposta-comunale-sulla-pubblicita": [],
            "4.3.4-contratti-di-accensione-mutui": [],
            "4.3.5-ruolo-diritti-sulle-pubbliche-affissioni": [],
            "4.3.6-ruolo-tarsu": [],
            "4.3.7-ruolo-cosap": [],
            "4.3.8-proventi-da-affitti-e-locazioni": [],
            "4.3.9-diritti-di-segreteria": [],
            "4.3.10-matrici-dei-bollettini-delle-entrate": [],
            "4.3.11-ricevute-dei-versamenti-in-banca-per-diritti-di-segreteria": [],
            "4.3.12-fatture-emesse-repertorio-annuale": [],
            "4.3.13-reversali-repertorio-annuale": [],
            "4.3.14-bollettari-vari-repertori-annuali": [],
            "4.3.15-ricevute-di-pagamenti-vari": []
        },
        "4.4-gestione-della-spesa-impegni-liquidazione-ordinazione-pagamento": {
            "4.4.1-impegni-di-spesa": [],
            "4.4.2-fatture-ricevute-repertorio-annuale": [],
            "4.4.3-atti-di-liquidazione-con-allegati-repertorio-annuale": [],
            "4.4.4-eventuali-copie-di-mandati-repertorio-annuale": [],
            "4.4.5-spese-postali": [],
            "4.4.6-tracciabilita-e-durc": []
        },
        "4.5-partecipazioni-finanziarie": {
            "4.5.1-gestione-delle-partecipazioni": []
        },
        "4.6-rendiconto-della-gestione-adempimenti-e-verifiche-contabili": {
            "4.6.1-rendiconto-della-gestione-conto-bilancio-patrimonio-economico": []
        },
        "4.7-adempimenti-fiscali-contributivi-assicurativi": {
            "4.7.1-modello-770": [],
            "4.7.2-ricevute-dei-versamenti-iva-irpef-ect": [],
            "4.7.3-pagamento-dei-premi-dei-contratti-assicurativi": []
        },
        "4.8-beni-immobili": {
            "4.8.1-inventario-dei-beni-immobili": [],
            "4.8.2-fascicoli-dei-beni-immobili": [],
            "4.8.3-concessione-di-occupazione-di-spazi-e-aree-pubbliche": [],
            "4.8.4-concessione-di-beni-del-demanio-statale": [],
            "4.8.5-concessioni-cimiteriali": [],
            "4.8.6-fascicoli-personali-dei-concessionari": []
        },
        "4.9-beni-mobili": {
            "4.9.1-inventario-dei-beni-mobili": [],
            "4.9.2-fascicoli-dei-beni-mobili": []
        },
        "4.10-economato": {
            "4.10.1-acquisizione-di-beni-e-servizi": [],
            "4.10.2-elenco-dei-fornitori": []
        },
        "4.11-oggetti-smarriti-e-recuperati": {
            "4.11.1-verbali-di-rinvenimento": [],
            "4.11.2-ricevute-di-riconsegna-ai-proprietari": [],
            "4.11.3-vendita-o-devoluzione": []
        },
        "4.12-tesoreria": {
            "4.12.1-giornale-di-cassa": [],
            "4.12.2-mandati-quietanziati": []
        },
        "4.13-concessionari-e-altri-incarichi-della-riscossione-delle-entrate": {
            "4.13.1-concessionari": []
        },
        "4.14-pubblicita-e-pubbliche-affissioni": {
            "4.14.1-autorizzazione-alla-pubblicita-stabile": [],
            "4.14.2-autorizzazione-alla-pubblicita-circoscritta": [],
            "4.14.3-richieste-di-affissione-con-allegati": []
        }
    },
    
    # TITOLO V - AFFARI GENERALI
    "titolo-05-affari-generali": {
        "5.1-contenzioso": {
            "5.1.1-fascicolo-di-causa-civili-amministrative-penali-tributarie": []
        },
        "5.2-responsabilita-civile-patrimoniale-verso-terzi-assicurazioni": {
            "5.2.1-contratti-assicurativi": [],
            "5.2.2-richieste-pratiche-di-risarcimento": []
        },
        "5.3-pareri-e-consulenze": {
            "5.3.1-pareri-e-consulenze-specifiche": []
        }
    },
    
    # TITOLO VI - PIANIFICAZIONE, GESTIONE DEL TERRITORIO
    "titolo-06-pianificazione-gestione-del-territorio": {
        "6.1-urbanistica-prg-variazioni": {
            "6.1.1-prg-piano-regolatore-generale": [],
            "6.1.2-pareri-su-piani-sovracomunali": [],
            "6.1.3-certificati-di-destinazione-urbanistica": [],
            "6.1.4-variante-al-prg": []
        },
        "6.2-urbanistica-strumenti-di-attuazione-del-prg": {
            "6.2.1-piani-particolareggiati-del-prg": [],
            "6.2.2-piani-di-lottizzazione": [],
            "6.2.3-piani-di-edilizia-economica-e-popolare-peep": [],
            "6.2.4-piano-particolareggiato-infrastrutture-stradali-ppis": [],
            "6.2.5-piano-di-riqualificazione-urbana-pru": [],
            "6.2.6-piano-insediamenti-produttivi-pip": [],
            "6.2.7-programma-integrato-riqualificazione-urbanistica-edilizia-ambientale-piruea": [],
            "6.2.8-programma-riqualificazione-urbana-e-sviluppo-sostenibile-del-territorio": [],
            "6.2.9-piano-di-assetto-territoriale-del-comune-pat": [],
            "6.2.10-piano-degli-interventi-pi": [],
            "6.2.11-piano-di-assetto-territoriale-intercomunale-pati": [],
            "6.2.12-piano-di-riqualificazione-urbana-e-di-sviluppo-sostenibile-prrust": []
        },
        "6.3-edilizia-privata": {
            "6.3.1-autorizzazione-edilizia-repertorio": [],
            "6.3.2-fascicoli-dei-richiedenti-le-autorizzazioni": [],
            "6.3.3-accertamento-e-repressione-degli-abusi": [],
            "6.3.4-denunce-e-relazioni-finali-delle-opere-in-cemento-armato": [],
            "6.3.5-abusi-edilizi": []
        },
        "6.4-edilizia-pubblica": {
            "6.4.1-costruzione-di-edilizia-popolare": []
        },
        "6.5-opere-pubbliche": {
            "6.5.1-realizzazione-di-opere-pubbliche": [],
            "6.5.2-manutenzione-ordinaria": [],
            "6.5.3-manutenzione-straordinaria": []
        },
        "6.6-catasto": {
            "6.6.1-catasto-terreni-mappe": [],
            "6.6.2-catasto-terreni-registri": [],
            "6.6.3-catasto-terreni-indice-alfabetico-dei-possessori": [],
            "6.6.4-catasto-terreni-estratti-catastali": [],
            "6.6.5-catasto-terreni-denunce-variazione-volture": [],
            "6.6.6-catasto-fabbricati-mappe": [],
            "6.6.7-catasto-fabbricati-registri": [],
            "6.6.8-catasto-fabbricati-indice-alfabetico-dei-possessori": [],
            "6.6.9-catasto-fabbricati-estratti-catastali": [],
            "6.6.10-catasto-fabbricati-denunce-variazione-volture": [],
            "6.6.11-richieste-di-visure-e-certificati": []
        },
        "6.7-viabilita": {
            "6.7.1-piano-urbano-del-traffico": [],
            "6.7.2-piano-urbano-della-mobilita": [],
            "6.7.3-autorizzazione-in-deroga": []
        },
        "6.8-servizio-idrico-integrato-luce-gas-trasporti-rifiuti-e-altri-servizi": {
            "6.8.1-approvvigionamento-idrico-organizzazione-funzionamento": [],
            "6.8.2-iniziative-a-favore-dell-ambiente": [],
            "6.8.3-distribuzione-dell-acqua": [],
            "6.8.4-produzione-di-energia-elettrica-o-altre-fonti-energetiche": [],
            "6.8.5-trasporti-pubblici": [],
            "6.8.6-vigilanza-sui-gestori-dei-servizi": [],
            "6.8.7-fascicoli-relativi-alle-irregolarita": [],
            "6.8.8-dichiarazione-di-conformita-degli-impianti": []
        },
        "6.9-ambiente-autorizzazioni-monitoraggio-controllo": {
            "6.9.1-valutazione-pareri-di-impatto-ambientale": [],
            "6.9.2-monitoraggi-della-qualita-delle-acque": [],
            "6.9.3-monitoraggi-della-qualita-dell-aria": [],
            "6.9.4-monitoraggi-della-qualita-dell-etere": [],
            "6.9.5-altri-eventuali-monitoraggi": [],
            "6.9.6-controlli-a-campione-sugli-impianti-termici-dei-privati": [],
            "6.9.7-fascicoli-relativi-alle-irregolarita": []
        },
        "6.10-protezione-civile-ed-emergenze": {
            "6.10.1-segnalazioni-preventive-di-condizioni-metereologiche-avverse": [],
            "6.10.2-addestramento-ed-esercitazioni-per-la-protezione-civile": [],
            "6.10.3-interventi-per-emergenze": []
        }
    },
    
    # TITOLO VII - SERVIZI ALLA PERSONA
    "titolo-07-servizi-alla-persona": {
        "7.1-diritto-allo-studio-e-servizi": {
            "7.1.1-concessione-di-borse-di-studio": [],
            "7.1.2-distribuzione-buoni-libro": [],
            "7.1.3-gestione-buoni-pasto-degli-iscritti-alle-scuole": [],
            "7.1.4-verbali-del-comitato-genitori-per-la-mensa": [],
            "7.1.5-azioni-di-promozione-sostegno-del-diritto-allo-studio": [],
            "7.1.6-gestione-mense-scolastiche": [],
            "7.1.7-gestione-trasporto-scolastico": []
        },
        "7.2-asilo-e-scuola-materna": {
            "7.2.1-domande-di-ammissione-all-asilo-nido": [],
            "7.2.2-graduatorie-di-ammissione": [],
            "7.2.3-funzionamento-dell-asilo": []
        },
        "7.3-promozione-sostegno-istituzioni-di-istruzione-e-loro-attivita": {
            "7.3.1-iniziative-specifiche": [],
            "7.3.2-registri-scolastici-prodotti-dalle-scuole-civiche": []
        },
        "7.4-orientamento-professionale-educazione-degli-adulti-meditazione": {
            "7.4.1-iniziative-specifiche": []
        },
        "7.5-istituti-culturali-biblioteche-teatri": {
            "7.5.1-funzionamento-delle-diverse-istituzioni-culturali": [],
            "7.5.2-verbali-degli-organi-di-gestione-degli-istituti-culturali": []
        },
        "7.6-attivita-ed-eventi-culturali": {
            "7.6.1-attivita-ordinarie-annuali": [],
            "7.6.2-eventi-culturali": [],
            "7.6.3-feste-civili-o-religiose": [],
            "7.6.4-iniziative-culturali": [],
            "7.6.5-prestiti-di-beni-culturali": []
        },
        "7.7-attivita-ed-eventi-sportivi": {
            "7.7.1-eventi-attivita-sportive": []
        },
        "7.8-pianificazione-accordi-strategici-volontariato-sociale": {
            "7.8.1-piano-sociale": [],
            "7.8.2-programmazione-per-settore": [],
            "7.8.3-accordi-con-differenti-soggetti": []
        },
        "7.9-prevenzione-recupero-e-reintegrazione-dei-soggetti-a-rischio": {
            "7.9.1-campagna-di-prevenzione": [],
            "7.9.2-interventi-di-recupero-e-reintegrazione-dei-soggetti-a-rischio": [],
            "7.9.3-ricognizione-dei-rischi": []
        },
        "7.10-informazione-consulenza-educazione-civica": {
            "7.10.1-funzionamento-attivita-delle-strutture": [],
            "7.10.2-iniziative-di-vario-tipo": []
        },
        "7.11-tutela-e-curatela-di-incapaci": {
            "7.11.1-interventi-per-le-persone-sottoposte-a-tutela-e-curatela": []
        },
        "7.12-assistenza-diretta-indiretta-benefici-economici": {
            "7.12.1-funzionalmente-attivita-delle-strutture": []
        },
        "7.13-attivita-ricettive-e-di-socializzazione": {
            "7.13.1-funzionamento-attivita-delle-strutture": []
        },
        "7.14-politiche-per-la-casa": {
            "7.14.1-assegnazione-degli-alloggi": [],
            "7.14.2-fascicolo-degli-assegnatari": []
        },
        "7.15-politiche-per-il-sociale": {
            "7.15.1-iniziative-specifiche": []
        }
    },
    
    # TITOLO VIII - ATTIVITA' ECONOMICHE
    "titolo-08-attivita-economiche": {
        "8.1-agricoltura-e-pesca": {
            "8.1.1-iniziative-specifiche": [],
            "8.1.2-dichiarazioni-raccolte-produzione": []
        },
        "8.2-artigianato": {
            "8.2.1-iniziative-specifiche": [],
            "8.2.2-autorizzazione-artigiani-repertorio": []
        },
        "8.3-industria": {
            "8.3.1-iniziative-specifiche": []
        },
        "8.4-commercio": {
            "8.4.1-iniziative-specifiche": [],
            "8.4.2-comunicazioni-dovute": [],
            "8.4.3-autorizzazioni-commerciali-repertorio": [],
            "8.4.4-autorizzazioni-artigianali-repertorio": [],
            "8.4.5-autorizzazioni-commerciali-repertorio-bis": [],
            "8.4.6-autorizzazioni-turistiche-repertorio": []
        },
        "8.5-fiere-e-mercati": {
            "8.5.1-iniziative-specifiche": []
        },
        "8.6-esercizi-turistici-e-strutture-ricettive": {
            "8.6.1-iniziative-specifiche": [],
            "8.6.2-autorizzazioni-turistiche-repertorio": []
        },
        "8.7-promozione-servizi": {
            "8.7.1-iniziative-specifiche": []
        }
    },
    
    # TITOLO IX - POLIZIA LOCALE E LA SICUREZZA PUBBLICA
    "titolo-09-polizia-locale-e-la-sicurezza-pubblica": {
        "9.1-prevenzione-ed-educazione-stradale": {
            "9.1.1-iniziative-specifiche-di-prevenzione": [],
            "9.1.2-corsi-di-educazione-stradale-nelle-scuole": []
        },
        "9.2-polizia-stradale": {
            "9.2.1-direttiva-a-disposizione": [],
            "9.2.2-organizzazione-del-servizio-di-pattugliamento": [],
            "9.2.3-verbali-di-accertamento-di-violazione-al-codice-della-strada": [],
            "9.2.4-accertamento-di-violazione-e-irrogazione-di-sanzione": [],
            "9.2.5-verbali-di-rilevazione-incidenti": [],
            "9.2.6-statistiche-delle-violazioni-e-degli-incidenti": [],
            "9.2.7-gestione-veicoli-rimossi": []
        },
        "9.3-informative": {
            "9.3.1-informative-su-persone-residenti-nel-comune": []
        },
        "9.4-sicurezza-e-ordine-pubblico": {
            "9.4.1-direttive-disposizione-generale": [],
            "9.4.2-servizio-ordinario-di-pubblica-sicurezza": [],
            "9.4.3-servizio-straordinario-di-pubblica-sicurezza-eventi-particolari": []
        }
    },
    
    # TITOLO X - TUTELA DELLA SALUTE
    "titolo-10-tutela-della-salute": {
        "10.1-salute-e-igiene-pubblica": {
            "10.1.1-emergenze-sanitarie": [],
            "10.1.2-misure-d-igiene-pubblica": [],
            "10.1.3-interventi-di-derattizzazione-di-zanzarizzazione-ect": [],
            "10.1.4-trattamenti-fitosanitari-e-di-disinfestazione": [],
            "10.1.5-autorizzazioni-sanitarie-repertorio-annuale": [],
            "10.1.6-autorizzazione-di-concessioni-di-agibilita-repertorio-annuale": [],
            "10.1.7-fascicoli-dei-richiedenti-autorizzazione-sanitarie": [],
            "10.1.8-fascicoli-dei-richiedenti-agibilita": []
        },
        "10.2-trattamenti-sanitari-obbligatori": {
            "10.2.1-tso-trattamento-sanitario-obbligatorio": [],
            "10.2.2-aso-accertamento-sanitario-obbligatorio": [],
            "10.2.3-fascicoli-personali-dei-soggetti-a-trattamenti": []
        },
        "10.3-farmacie": {
            "10.3.1-istituzioni-di-farmacie": [],
            "10.3.2-funzionamento-delle-farmacie": []
        },
        "10.4-zooprofilassi-veterinaria": {
            "10.4.1-fascicolo-relativi-a-epizoozie-epidemie-animali": []
        },
        "10.5-randagismo-animale-e-ricoveri": {
            "10.5.1-gestione-dei-ricoveri-e-degli-eventi-connessi": [],
            "10.5.2-verbali-degli-accertamenti-nei-diversi-settori": []
        }
    },
    
    # TITOLO XI - SERVIZI DEMOGRAFICI
    "titolo-11-servizi-demografici": {
        "11.1-stato-civile": {
            "11.1.1-registro-dei-nati": [],
            "11.1.2-registro-dei-morti": [],
            "11.1.3-repertorio-dei-matrimoni": [],
            "11.1.4-registro-di-cittadinanza": [],
            "11.1.5-altri-allegati-per-registrazioni": [],
            "11.1.6-registro-della-popolazione": [],
            "11.1.7-registro-della-distribuzione-topografica-delle-tombe-con-schede-onomastiche": [],
            "11.1.8-atti-per-annotazioni-sui-registri-di-stato-civile": [],
            "11.1.9-comunicazione-dei-nati-all-agenzia-delle-entrate": []
        },
        "11.2-anagrafe-certificazioni": {
            "11.2.1-apr4-iscrizioni-anagrafiche": [],
            "11.2.2-aire-anagrafe-italiani-residenti-all-estero": [],
            "11.2.3-richieste-certificati": [],
            "11.2.4-corrispondenza-con-le-altre-amministrazioni": [],
            "11.2.5-cartellini-per-carta-d-identita": [],
            "11.2.6-carta-d-identita-scaduta-e-riconsegnate": [],
            "11.2.7-cambio-di-abitazione-residenza": [],
            "11.2.8-cancellazioni": [],
            "11.2.9-carteggio-con-la-corte-d-appello-per-la-formazione-degli-albi-giudici-popolari": [],
            "11.2.10-registro-della-popolazione-su-base-di-dati": []
        },
        "11.3-censimenti": {
            "11.3.1-schedoni-statistici-del-censimento": [],
            "11.3.2-atti-preparatori-organizzativi": []
        },
        "11.4-polizia-mortuaria-cimiteri": {
            "11.4.1-registri-di-seppellimento": [],
            "11.4.2-registri-di-tumulazione": [],
            "11.4.3-registri-di-esumazioni": [],
            "11.4.4-registri-di-estumulazione": [],
            "11.4.5-registri-di-cremazione": [],
            "11.4.6-registri-della-distribuzione-topografica-delle-tombe-con-schede-onomastica": [],
            "11.4.7-trasferimento-delle-salme": []
        }
    },
    
    # TITOLO XII - ELEZIONI, INIZIATIVE POPOLARI
    "titolo-12-elezioni-iniziative-popolari": {
        "12.1-albi-elettorali": {
            "12.1.1-albo-dei-presidenti-di-seggio": [],
            "12.1.2-albo-degli-scrutatori": [],
            "12.1.3-verbali-della-commissione-elettorale-comunale": [],
            "12.1.4-verbali-dei-presidenti-di-seggio": []
        },
        "12.2-liste-elettorali": {
            "12.2.1-liste-generali": [],
            "12.2.2-liste-sezionali": [],
            "12.2.3-verbali-della-commissione-elettorale-mandamentale": [],
            "12.2.4-schede-dello-schedario-generale": [],
            "12.2.5-schede-degli-schedari-sezionali": [],
            "12.2.6-fascicoli-personali-degli-elettori": [],
            "12.2.7-elenchi-recanti-le-proposte-di-variazione-delle-liste-elettorali": [],
            "12.2.8-carteggio-concernente-la-tenuta-e-la-rilevazione-delle-liste-elettorali": []
        },
        "12.3-elezioni": {
            "12.3.1-convocazione-dei-comizi-elettorali": [],
            "12.3.2-presentazione-delle-liste-manifesto": [],
            "12.3.3-presentazione-delle-liste-carteggio": [],
            "12.3.4-atti-relativi-alla-costituzione-e-arredamento-dei-seggi": [],
            "12.3.5-verbali-dei-presidenti-di-seggio": [],
            "12.3.6-schede": [],
            "12.3.7-pacchi-scorta-elezioni": [],
            "12.3.8-certificati-elettorali-non-ritirati": [],
            "12.3.9-istruzioni-elettorali-a-stampa": []
        },
        "12.4-referendum": {
            "12.4.1-atti-preparatori": [],
            "12.4.2-atti-relativi-alla-costituzione-arredamento-dei-seggi": [],
            "12.4.3-verbali-dei-presidenti-di-seggio": [],
            "12.4.4-schede": []
        },
        "12.5-istanze-petizione-iniziative-popolari": {
            "12.5.1-raccolta-di-firme-per-referendum-previste-dallo-statuto": []
        }
    },
    
    # TITOLO XIII - AFFARI MILITARI
    "titolo-13-affari-militari": {
        "13.1-leva-il-servizio-civile-sostitutivo": {
            "13.1.1-lista-di-leva": [],
            "13.1.2-liste-degli-eliminati-esentati": []
        },
        "13.2-ruoli-matricolari": {
            "13.2.1-ruoli-matricolari": []
        },
        "13.3-caserme-alloggio-servizi-militari": {
            "13.3.1-procedimenti-specifici": []
        },
        "13.4-requisiti-per-utilita-militare": {
            "13.4.1-procedimenti-specifici": []
        }
    },
    
    # TITOLO XIV - OGGETTI DIVERSI
    "titolo-14-oggetti-diversi": {
        "14.1-varie": {
            "14.1.1-documenti-non-riferibili-ad-altre-classi": []
        }
    }
}

# ==============================================================================
# 3. Funzioni MinIO
# ==============================================================================

def create_minio_client():
    """Crea e restituisce il client MinIO"""
    try:
        # MinIO client richiede l'endpoint senza http/https
        client = Minio(
            MINIO_ENDPOINT,
            access_key=MINIO_ACCESS_KEY,
            secret_key=MINIO_SECRET_KEY,
            secure=MINIO_SECURE
        )
        print(f"✓ Connesso a MinIO: {MINIO_ENDPOINT}")
        return client
    except Exception as e:
        print(f"✗ Errore connessione MinIO: {e}")
        return None

def create_bucket_if_not_exists(client, bucket_name):
    """Crea un bucket se non esiste"""
    bucket_name_lower = bucket_name.lower()
    try:
        if not client.bucket_exists(bucket_name_lower):
            client.make_bucket(bucket_name_lower)
            print(f"  ✓ Bucket creato: {bucket_name_lower}")
            return True
        else:
            print(f"  ℹ Bucket già esistente: {bucket_name_lower}")
            return False
    except S3Error as e:
        print(f"  ✗ Errore creazione bucket {bucket_name_lower}: {e}")
        return False

def create_folder_structure(client, bucket_name, folder_path):
    """Crea una struttura di cartelle (simulata con un file .gitkeep)"""
    try:
        # MinIO richiede un file marker per creare "cartelle"
        folder_marker = f"{folder_path}/.gitkeep"
        client.put_object(
            bucket_name.lower(),
            folder_marker,
            data=b'',
            length=0,
            content_type='application/octet-stream'
        )
        return True
    except S3Error as e:
        # Ignora l'errore se la cartella esiste già
        if 'Key already exists' in str(e):
             return True
        print(f"    ✗ Errore creazione cartella {folder_path}: {e}")
        return False

def create_complete_structure(client):
    """
    Crea la struttura completa del Titolario su MinIO.
    Logica: TITOLO (Bucket) -> ANNO -> CLASSE -> SOTTOCLASSE -> [SUB-SOTTOCLASSE]
    """
    print("\n" + "="*60)
    print("CREAZIONE STRUTTURA TITOLARIO COMPLETA SU MINIO")
    print("="*60 + "\n")

    anno_corrente = datetime.now().year
    
    # Cartelle finali comuni per la documentazione
    # Queste vengono aggiunte al livello di classificazione più profondo
    final_subfolders = ["documenti", "fascicoli", "repertori"]

    for bucket_name, classi in TITOLARIO.items():
        bucket_name_lower = bucket_name.lower()
        
        print(f"\n📁 TITOLO: {bucket_name_lower.upper()}")
        print("-" * 60)
        
        # 1. Crea il bucket principale (Titolo)
        create_bucket_if_not_exists(client, bucket_name_lower)
            
        # 2. Gestione Speciale: Fascicoli del Personale (Solo per Titolo 03)
        if bucket_name == "titolo-03-risorse-umane":
            # Questa serie contiene i fascicoli individuali (serie Attivi e Cessati)
            personale_attivi_path = f"fascicoli-personale-serie-attivi/{anno_corrente}"
            create_folder_structure(client, bucket_name_lower, personale_attivi_path)
            print(f"    ✓ SERIE SPECIALE: {personale_attivi_path}")
            
            personale_cessati_path = f"fascicoli-personale-serie-cessati/{anno_corrente}"
            create_folder_structure(client, bucket_name_lower, personale_cessati_path)
            print(f"    ✓ SERIE SPECIALE: {personale_cessati_path}")


        # 3. Iterazione su Classi, Sottoclassi e Sub-Sottoclassi
        for classe_name, sottoclassi in classi.items():
            class_base_path = f"{anno_corrente}/{classe_name}"
            # Crea la cartella di Classe (Es. 2025/1.1-legislazione...)
            create_folder_structure(client, bucket_name_lower, class_base_path)
            # print(f"    ✓ Classe: {class_base_path}") # Commentato per ridurre l'output

            for sottoclasse_name, sub_sottoclassi in sottoclassi.items():
                sottoclasse_path = f"{class_base_path}/{sottoclasse_name}"
                # Crea la cartella di Sottoclasse (Es. 2025/1.1-legislazione/1.1.1-normativa-varie)
                create_folder_structure(client, bucket_name_lower, sottoclasse_path)
                
                # Determina il percorso finale dove aggiungere le cartelle di documentazione
                target_path = sottoclasse_path
                has_sub_sub = False

                if sub_sottoclassi:
                    has_sub_sub = True
                    for sub_sottoclasse_name in sub_sottoclassi:
                        # Crea la cartella di Sub-Sottoclasse (4° Livello)
                        target_path = f"{sottoclasse_path}/{sub_sottoclasse_name}"
                        create_folder_structure(client, bucket_name_lower, target_path)
                        
                        # Aggiungiamo le cartelle finali al 4° livello
                        for final_subfolder in final_subfolders:
                            final_path = f"{target_path}/{final_subfolder}"
                            create_folder_structure(client, bucket_name_lower, final_path)
                        print(f"    -> Creazione 4° livello: {target_path}")

                if not has_sub_sub:
                    # Se il livello più profondo è il 3° (Sottoclasse), aggiungi qui le cartelle finali
                    for final_subfolder in final_subfolders:
                        final_path = f"{sottoclasse_path}/{final_subfolder}"
                        create_folder_structure(client, bucket_name_lower, final_path)
                    print(f"    -> Creazione 3° livello: {sottoclasse_path}")


# ==============================================================================
# 4. Funzione Principale
# ==============================================================================

def main():
    """Funzione principale"""
    # Installa la libreria minio
    try:
        import subprocess
        # Assicurati che la libreria MinIO sia installata all'interno del container
        subprocess.check_call(['pip', 'install', 'minio'])
    except Exception as e:
        # Se pip è già eseguito nella shell non importa
        pass

    print("\n🚀 INIZIALIZZAZIONE ARCHIVIO COMUNALE SU MINIO CON TITOLARIO COMPLETO")

    client = create_minio_client()
    if not client:
        print("\n❌ Impossibile procedere. Controllare MINIO_ENDPOINT, ACCESS_KEY e SECRET_KEY.")
        return

    create_complete_structure(client)

    print("\n" + "="*60)
    print("✅ STRUTTURA TITOLARIO COMPLETATA")
    print("="*60)
    print(f"\nBucket creati (Titoli): {len(TITOLARIO)}")
    print(f"Anno di riferimento: {datetime.now().year}")

if __name__ == "__main__":
    main()

3. Guida all'Esecuzione
Passo 3.1: Trova il Nome della Rete Docker
Esegui questo comando nella directory del tuo docker-compose.yml per identificare il nome completo della rete interna (necessario per iniettare il container temporaneo):
docker network ls | grep backend

> Esempio di Output: myproject_backend
> Memorizza il nome completo (ad esempio: <IL_TUO_NOME_PROGETTO>_backend).
> 
Passo 3.2: Esegui il Container Temporaneo
Il comando seguente avvia un container Python, collega lo script, lo inietta nella rete (--network) e imposta le credenziali per connettersi a MinIO (minio:9000).
Eseguilo nel tuo terminale, sostituendo i placeholder con i tuoi valori:
# Sostituisci:
# 1. <NOME_COMPLETO_DELLA_RETE_BACKEND> (es: myproject_backend)
# 2. <MINIO_ROOT_USER_VALUE> (il tuo utente MinIO)
# 3. <MINIO_ROOT_PASSWORD_VALUE> (la tua password MinIO)

docker run --rm \
  --name titolario-initializer-temp \
  -v $(pwd)/create_titolario_minio.py:/app/script.py \
  --network <NOME_COMPLETO_DELLA_RETE_BACKEND> \
  -e MINIO_ENDPOINT="minio:9000" \
  -e MINIO_ACCESS_KEY="<MINIO_ROOT_USER_VALUE>" \
  -e MINIO_SECRET_KEY="<MINIO_ROOT_PASSWORD_VALUE>" \
  -e MINIO_SECURE="false" \
  python:3.11-slim \
  sh -c "pip install minio && python /app/script.py"

3.3. Risultato e Pulizia
 * Esecuzione: Il terminale mostrerà i log dettagliati della creazione dei 14 Bucket (Titoli) e di tutte le Classi/Sottoclassi al loro interno.
 * Pulizia: Grazie all'opzione --rm, il container titolario-initializer-temp viene automaticamente rimosso una volta che lo script ha terminato il suo lavoro, lasciando inalterato il tuo docker-compose.yml.
 * Verifica: Controlla la console di MinIO (solitamente su 127.0.0.1:9001) per vedere la struttura completa creata per l'anno corrente.
