# Conti di Casa — Migrazione a Netlify (Blobs + Functions)

Struttura del progetto:

```
mio-progetto/
├── public/
│   └── index.html          (frontend, modificato per usare la Function)
├── netlify/
│   └── functions/
│       └── api.js          (backend: legge/scrive su Netlify Blobs)
├── package.json
├── netlify.toml             (dice a Netlify dove sono public/ e functions/)
└── README.md
```

## Cosa è cambiato

- `index.html`: le funzioni `loadAll`, `saveSettings`, `saveMovements` non usano più
  `window.storage.get/set` ma chiamano `/.netlify/functions/api?key=...` con `fetch`.
- `netlify/functions/api.js`: nuova Netlify Function che legge/scrive su **Netlify Blobs**
  (store chiamato `conti-di-casa-data`), esposta come endpoint `/.netlify/functions/api`.
- Corretto un piccolo refuso nella bozza originale: lo status code per "metodo non
  consentito" è `405` (non `450`, che non è un codice HTTP valido).

## Come pubblicare

1. **Installa le dipendenze** (nella cartella `mio-progetto`):
   ```
   npm install
   ```

2. **Prova in locale** (richiede Netlify CLI, già in devDependencies):
   ```
   npx netlify dev
   ```
   Questo avvia sia il frontend che le Functions in locale, con storage Blobs
   in locale già emulato da Netlify CLI. Apri l'URL che ti stampa in console
   (di solito `http://localhost:8888`).

3. **Deploy**:
   - Opzione A — collega il repo Git a Netlify (dashboard → "Add new site" →
     "Import an existing project") e lascia che rilevi automaticamente
     `netlify.toml`.
   - Opzione B — deploy manuale da CLI:
     ```
     npx netlify deploy --prod
     ```

   Netlify Blobs è già abilitato di default sui siti Netlify: non serve
   creare un database esterno né configurare chiavi API.

## Note

- Ogni "tabella" (settings / expenses) è salvata come una singola chiave nello
  store Blobs `conti-di-casa-data`, esattamente come prima veniva salvata come
  singola chiave in `window.storage` — quindi la logica dell'app non cambia,
  cambia solo dove i dati vivono fisicamente (ora persistono sul cloud Netlify
  invece che nel browser).
- Se in futuro ti serve un vero database relazionale (es. per query più
  complesse), la stessa Function può essere riscritta per parlare con
  Supabase o Neon Postgres invece che con Blobs — l'interfaccia verso il
  frontend (`GET/POST /.netlify/functions/api?key=...`) resterebbe identica.
