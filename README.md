# 🏨 Hotel Dashboard

Dashboard collaborativa per la gestione dei flussi lavorativi di un hotel: mansioni divise per turno (mattina/pomeriggio/sera) e area (sala/bar/check), con tracking di chi ha completato cosa, storico giornaliero, pannello admin e sincronizzazione real-time tra tutti i device.

**Stack**: HTML/CSS/JS singolo file + React via CDN + Firebase Firestore. Zero build, zero dipendenze locali, deploy diretto su GitHub Pages.

---

## 🚀 Setup completo (≈10-15 minuti)

### Parte 1 — Crea il progetto Firebase

1. Vai su [console.firebase.google.com](https://console.firebase.google.com) e accedi con il tuo account Google.
2. Clicca **"Aggiungi progetto"**.
3. Nome del progetto: es. `hotel-dashboard`. Avanti.
4. **Disattiva Google Analytics** (non serve). Crea progetto.
5. Attendi qualche secondo, poi clicca **"Continua"**.

### Parte 2 — Crea il database Firestore

1. Nel menu a sinistra: **Build → Firestore Database**.
2. Clicca **"Crea database"**.
3. Modalità: scegli **"Avvia in modalità test"**. Continua.
4. Località: scegli **`eur3 (europe-west)`** o **`europe-west1`** (più vicina all'Italia). Crea.

### Parte 3 — Imposta le regole di sicurezza

1. Nella sezione Firestore, clicca la tab **"Regole"** in alto.
2. Cancella tutto il contenuto e incolla **esattamente** quello che trovi nel file `firestore.rules` di questa repo.
3. Clicca **"Pubblica"**.

### Parte 4 — Registra l'app web

1. Nella home del progetto Firebase, clicca l'icona **`</>`** (Web) per aggiungere un'app web.
2. Soprannome app: es. `Hotel Dashboard`. **NON** spuntare "Configura anche Firebase Hosting". Registra app.
3. Firebase ti mostra un blocco di codice con `const firebaseConfig = {...}`. **Copia tutti i valori** (apiKey, authDomain, projectId, ecc.).

### Parte 5 — Configura `index.html`

Apri `index.html` (qui sulla repo, click sul file → matita per modificare). Cerca la sezione:

```javascript
const firebaseConfig = {
  apiKey: "PASTE_YOUR_API_KEY",
  authDomain: "PASTE_YOUR_AUTH_DOMAIN",
  projectId: "PASTE_YOUR_PROJECT_ID",
  storageBucket: "PASTE_YOUR_STORAGE_BUCKET",
  messagingSenderId: "PASTE_YOUR_SENDER_ID",
  appId: "PASTE_YOUR_APP_ID"
};
```

Sostituisci ogni `PASTE_YOUR_...` con i valori che ti ha dato Firebase. Salva (commit).

### Parte 6 — Pubblica con GitHub Pages

1. Sulla repo GitHub, vai in **Settings → Pages** (sidebar a sinistra).
2. **Source**: seleziona `Deploy from a branch`.
3. **Branch**: scegli `main` e cartella `/ (root)`. Salva.
4. Aspetta 1-2 minuti. In alto comparirà:
   > **Your site is live at `https://TUO-USERNAME.github.io/NOME-REPO/`**

Quello è il link da condividere con i dipendenti.

---

## 📱 Aggiungere l'icona alla home dello smartphone

I dipendenti possono "installare" l'app come icona sulla home:

**iPhone (Safari)**: apri il link → tocca l'icona Condividi (in basso) → "Aggiungi a Home".

**Android (Chrome)**: apri il link → menu ⋮ in alto a destra → "Aggiungi a schermata Home".

L'app si aprirà come un'app nativa, senza barre del browser.

---

## 🔐 Accesso

- **Dipendenti**: aprono il link, selezionano il loro nome dalla lista. Fine.
- **Admin**: dal login → "Accesso amministratore" → PIN (default `0000`, **cambialo subito** dalla sezione Impostazioni).

L'admin può:
- Aggiungere/rimuovere mansioni e dipendenti
- Vedere lo storico di tutti i giorni passati
- Cambiare il PIN
- Resettare le mansioni della giornata corrente

---

## ⚠️ Note di sicurezza

L'accesso è **pubblico**: chiunque conosca il link può entrare e segnare task come fatte. Le regole Firestore sono permissive per scelta. 

**Non condividere il link al di fuori del tuo staff.** Se il link finisse in mano sbagliata, qualcuno potrebbe spuntare task arbitrariamente o cambiarle dal pannello admin (se conosce il PIN).

Per limitare l'accesso in futuro, le opzioni sono:
- **Login Google obbligatorio**: abilita Firebase Authentication, cambia le regole `if true` in `if request.auth != null`. Ogni dipendente fa login con account Google.
- **Password unica condivisa**: aggiungere una schermata di login con password al posto della selezione diretta del nome.

Chiedi se vuoi implementare una di queste.

---

## 🧪 Limiti del piano gratuito Firebase

Il piano **Spark (gratuito)** include:
- 50.000 letture/giorno
- 20.000 scritture/giorno
- 1 GB di storage

Per un hotel con 8 dipendenti che spuntano ~50 task/giorno: **~500 scritture/giorno**. Sei al **2,5% del limite**. Resterà gratuito per anni.

---

## 🛠️ Architettura tecnica

- **Frontend**: HTML + React (via CDN), Tailwind, JSX compilato al volo da Babel Standalone. Zero build step.
- **Backend**: Firebase Firestore. Due collection:
  - `app/config` — un solo doc con `employees`, `tasks`, `pin`.
  - `days/{YYYY-MM-DD}` — un doc per giorno con `{taskId: {done, doneBy, doneById, doneAt}}`.
- **Sync**: `onSnapshot` di Firestore → aggiornamenti push automatici a tutti i device in <2 sec.
- **Race condition**: eliminata. Ogni task è un campo separato, le scritture usano `merge: true` su singoli campi → due dipendenti possono spuntare task diverse simultaneamente senza conflitti.

---

## 🆘 Troubleshooting

**"Configurazione Firebase mancante"**: non hai sostituito i `PASTE_YOUR_*` in `index.html`.

**"Errore di connessione" / "permission-denied"**: le regole Firestore non sono state pubblicate, o sono ancora quelle di default. Vai su Firebase Console → Firestore → Regole, incolla il contenuto di `firestore.rules` e pubblica.

**Lo schermo resta bianco**: apri la console del browser (F12) e controlla gli errori. Spesso è una virgola mancante in `firebaseConfig` dopo la modifica.

**Le modifiche non si propagano tra device**: controlla che entrambi i device siano connessi a internet. Firestore mette in cache offline, ma sincronizza solo con connessione attiva.
