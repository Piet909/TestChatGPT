# Gestionale Casa + Airtable + GitHub: guida pratica

Questa guida ti spiega come creare un'applicazione (hostata su GitHub) per gestire informazioni e link utili all'acquisto casa, con **Airtable** come database, e con una funzione che suggerisce la casa migliore in base a **qualità/prezzo**.

## 1) Chiarimento importante: “interrogarla da GitHub”

Hai 2 strade possibili:

1. **Repository + Actions (senza backend sempre acceso)**
   - Usi GitHub come “contenitore” del codice.
   - Esegui script tramite GitHub Actions (manuale/schedulato).
   - Va bene per report periodici, non ideale per una web app interattiva in tempo reale.

2. **Repository + Web App deployata (consigliato)**
   - Codice su GitHub.
   - App online su Vercel/Render/Railway.
   - Da GitHub fai deploy CI/CD automatico.
   - È la scelta migliore per un gestionale consultabile sempre.

> Ti consiglio la strada 2.

## 2) Architettura consigliata (semplice e scalabile)

- **Frontend**: Next.js (React)
- **Backend API**: route API di Next.js (o server separato Node/Express)
- **Database**: Airtable (una o più tabelle)
- **Auth**: GitHub OAuth (opzionale, ma utile)
- **Deploy**: Vercel collegato al repo GitHub

Flusso:
1. Inserisci/aggiorni case nel gestionale.
2. L’app salva su Airtable.
3. L’app calcola uno score “qualità/prezzo”.
4. Mostra ranking e “miglior casa” secondo i tuoi pesi.

## 3) Struttura dati in Airtable

Crea una base `GestionaleCase` con tabella `Case`.

Campi minimi suggeriti:
- `Titolo` (single line text)
- `Prezzo` (currency)
- `Mq` (number)
- `ZonaScore` (number 1-10)
- `StatoImmobileScore` (number 1-10)
- `ServiziViciniScore` (number 1-10)
- `ClasseEnergeticaScore` (number 1-10)
- `LinkAnnuncio` (url)
- `Note` (long text)
- `QualitaPrezzoScore` (formula o calcolato da app)

Opzionale:
- `MutuoStimatoMensile`
- `SpeseCondominiali`
- `DistanzaLavoroMinuti`

## 4) Algoritmo qualità/prezzo (versione base)

Esempio semplice:

- `qualita = 0.30*Zona + 0.30*Stato + 0.20*Servizi + 0.20*ClasseEnergetica`
- `prezzo_normalizzato = Prezzo / Mq`
- `score_finale = qualita * 100 / prezzo_normalizzato`

Più alto è `score_finale`, migliore è il rapporto qualità/prezzo.

### Versione personalizzabile

Aggiungi pesi configurabili da UI:
- peso zona
- peso stato
- peso servizi
- peso efficienza energetica
- penalità per prezzo alto

In questo modo l’app si adatta alle tue priorità.

## 5) Setup tecnico passo-passo

## Passo A — Crea repo GitHub

- Crea repository (es. `gestionale-casa`).
- In locale:

```bash
npx create-next-app@latest gestionale-casa
cd gestionale-casa
npm install airtable zod
```

## Passo B — Configura Airtable API

1. Crea Personal Access Token in Airtable.
2. Prendi `BASE_ID` e nome tabella.
3. Metti variabili in `.env.local`:

```env
AIRTABLE_TOKEN=...
AIRTABLE_BASE_ID=...
AIRTABLE_TABLE_NAME=Case
```

## Passo C — Crea API route per leggere/scrivere case

- `GET /api/houses` per lista case
- `POST /api/houses` per creare nuova casa
- `PATCH /api/houses/:id` per aggiornare

Valida input con `zod`.

## Passo D — Crea dashboard gestionale

Pagine minime:
- Lista case (tabella con filtri)
- Dettaglio casa
- Form inserimento/modifica
- Classifica qualità/prezzo

## Passo E — Deploy e CI/CD

- Collega repo a Vercel.
- Ogni push su `main` fa deploy automatico.
- Configura env vars su Vercel.

## 6) “Applicazione su GitHub” vs “GitHub App”

Se intendi una vera **GitHub App** (integrazione GitHub):
- Ti serve solo per autenticazione/permessi su GitHub.
- Non è necessaria per il tuo gestionale immobiliare.

Per il tuo caso basta:
- Repo GitHub + web app deployata.
- (Opzionale) login con GitHub OAuth per accesso utenti.

## 7) Roadmap consigliata (MVP in 2 settimane)

Settimana 1:
- Modello dati Airtable
- CRUD case
- UI base dashboard

Settimana 2:
- Algoritmo scoring qualità/prezzo
- Filtri avanzati (budget, mq, zona)
- Deploy + test utenti

## 8) Feature utili dopo MVP

- Confronto 2–3 immobili affiancati
- Simulazione mutuo (rata, TAEG, anticipo)
- Alert: nuove case sotto una soglia prezzo/mq
- Integrazione mappe per tempi di percorrenza
- Export PDF report per visita immobile

## 9) Sicurezza e buone pratiche

- Non esporre mai token Airtable nel frontend.
- Chiamate Airtable solo da backend.
- Aggiungi rate limiting alle API.
- Logga errori e usa monitoraggio (Sentry o simile).

## 10) Se vuoi, posso prepararti subito uno starter

Posso generarti nel prossimo step:
- struttura cartelle Next.js
- API `/api/houses`
- util per calcolo score qualità/prezzo
- pagina dashboard con tabella + ordinamento

Così parti direttamente con un progetto già funzionante.

## 11) Non vedi il commit su GitHub? Controlla push e remote

Se il commit esiste in locale ma non su GitHub, di solito manca il collegamento al repository remoto o il push.

Comandi utili:

```bash
git status
git log --oneline -5
git remote -v
git branch
```

Se `git remote -v` non mostra `origin`, collega il repo:

```bash
git remote add origin https://github.com/<tuo-utente>/<tuo-repo>.git
git push -u origin <nome-branch>
```

Se `origin` c'è già:

```bash
git push
```

Poi ricarica la pagina del repository GitHub e verifica il branch corretto.
