# Telegram File Sender per Hermes Agent

Skill/plugin Hermes per inviare file locali su Telegram in modo affidabile, evitando i blocchi silenziosi del gateway quando un file non si trova in una directory media consentita.

Questa skill nasce per l'uso quotidiano con Hermes Agent su Telegram: PDF, DOCX, XLSX, ZIP, immagini e altri allegati locali vengono prima copiati in una cache sicura e poi inviati come allegati nativi Telegram.

## Nota personale

Sono un local SEO, non un programmatore.

Per quanto capiti di lavorare con HTML, CSS, automazioni e strumenti AI, il mio mestiere principale è un altro. Questa skill nasce da un problema concreto che ho incontrato usando Hermes con Telegram: inviare file locali, soprattutto PDF e documenti, senza incappare in invii fantasma che sembrano perdersi nel nulla.

L'ho pubblicata nella speranza che possa essere utile a chi si trova nella mia stessa situazione. Se all'interno della skill troverete errori, soluzioni inefficienti o vere e proprie castronerie, chiedo venia in anticipo 😅

Come sempre, commenti, correzioni e critiche costruttive sono ben accetti. 🤗

## Funzionamento

Hermes può inviare media a Telegram tramite `MEDIA:/percorso/file`, ma il gateway accetta solo file posizionati in cartelle autorizzate. Se il file si trova altrove — ad esempio in una cartella progetto, in `_inbox/raw`, in `Downloads` o in una directory cliente — il gateway può scartarlo senza un errore evidente.

La skill risolve il problema con un flusso semplice:

1. **Individua il file locale** da inviare.
2. **Copia il file nella cache documenti consentita** di Hermes:
   - `~/.hermes/cache/documents/telegram-sender/`
3. **Normalizza il nome file** per ridurre problemi con Telegram o con il gateway:
   - niente spazi;
   - niente caratteri speciali;
   - nome breve;
   - estensione originale mantenuta.
4. **Invia il file a Telegram**:
   - documenti/PDF/ZIP/XLSX via Telegram Bot API `sendDocument`;
   - immagini anche via `MEDIA:` quando appropriato.
5. **Opzionalmente pulisce la cache** dopo l'invio.

## Cosa risolve

- Invio affidabile di PDF e documenti da percorsi locali non autorizzati.
- Evita il classico problema del `MEDIA:` che non consegna nulla perché il file è fuori dalla media cache.
- Fornisce una procedura ripetibile per allegati destinati a Telegram.
- Mantiene il file come allegato nativo Telegram, non come link esterno.

## Requisiti

- Hermes Agent installato e configurato.
- Gateway Telegram attivo.
- Variabile `TELEGRAM_BOT_TOKEN` presente in:
  - `~/.hermes/.env`
- Chat ID Telegram conosciuto. Esempio placeholder:
  - `<TELEGRAM_CHAT_ID>`

> Nota: il valore del `CHAT_ID` va adattato al proprio ambiente.

## Installazione del plugin/skill

### Metodo consigliato: clone del repository

```bash
cd /tmp
git clone https://github.com/dexter7wolf/telegram-file-sender.git
mkdir -p ~/.hermes/skills/telegram-file-sender
cp telegram-file-sender/SKILL.md ~/.hermes/skills/telegram-file-sender/SKILL.md
```

Poi riavvia Hermes o apri una nuova sessione, così la skill viene riletta dal loader.

### Metodo alternativo: symlink per sviluppo

Utile se vuoi modificare la skill e mantenerla aggiornata dal repository Git:

```bash
git clone https://github.com/dexter7wolf/telegram-file-sender.git ~/telegram-file-sender
mkdir -p ~/.hermes/skills
ln -sfn ~/telegram-file-sender ~/.hermes/skills/telegram-file-sender
```

### Verifica installazione

In una nuova sessione Hermes, controlla che la skill sia visibile:

```bash
hermes skills list | grep telegram-file-sender
```

Oppure chiedi all'agente di usare la skill `telegram-file-sender` quando deve inviare file locali su Telegram.

## Uso rapido

### 1. Copia il file nella cache documenti

```bash
mkdir -p ~/.hermes/cache/documents/telegram-sender
cp /path/to/file.pdf ~/.hermes/cache/documents/telegram-sender/file.pdf
```

### 2. Invio via Telegram Bot API

```bash
BOT_TOKEN=$(grep TELEGRAM_BOT_TOKEN ~/.hermes/.env | cut -d= -f2)
CHAT_ID=<TELEGRAM_CHAT_ID>

curl -F "document=@${HOME}/.hermes/cache/documents/telegram-sender/file.pdf" \
  "https://api.telegram.org/bot${BOT_TOKEN}/sendDocument" \
  -F "chat_id=${CHAT_ID}" \
  -F "caption=file.pdf"
```

### 3. Invio immagini via MEDIA

Per immagini già copiate nella cache:

```text
MEDIA:${HOME}/.hermes/cache/documents/telegram-sender/immagine.png
```

## Limiti noti

- Telegram Bot API accetta un file per chiamata `curl`.
- I limiti di upload dipendono dalle regole Telegram Bot API.
- I percorsi sono documentati con `~` o `${HOME}` per non esporre directory personali.
- In caso di profili Hermes multipli, controllare la `.env` del profilo corretto.

## Struttura repository

```text
telegram-file-sender/
├── README.md   # Documentazione, installazione, funzionamento e changelog
└── SKILL.md    # Skill Hermes vera e propria
```

## Changelog

### 1.0.1 - 2026-05-27

- Anonimizzati Chat ID e percorsi locali sensibili nella documentazione.
- Aggiunta nota personale sul contesto dell'autore e apertura a feedback costruttivi.

### 1.0.0 - 2026-05-27

- Prima pubblicazione su GitHub.
- Aggiunta procedura per copiare file arbitrari nella cache documenti Hermes.
- Aggiunto invio documenti/PDF/ZIP/XLSX via Telegram Bot API `sendDocument`.
- Documentato uso di `MEDIA:` per immagini già in percorso consentito.
- Documentati errori comuni: file fuori cache, silent drop, path non consentiti.

## Ringraziamenti

Un ringraziamento speciale a **Dagodom** per avermi fornito la sua skill di invio file PDF su Telegram. È stata il punto di partenza da cui ho realizzato e adattato questa skill per il mio workflow con Hermes.

A volte parlare con gli altri e condividere anche una piccola conoscenza, per quanto semplice, può bastare a sbloccarti, a darti l'ispirazione giusta o ad accendere un'idea che fino a quel momento avevi ignorato. Sicuramente, senza il codice che Dagodom mi ha gentilmente condiviso, ci avrei messo molto più tempo e molti più tentativi prima di arrivare a una soluzione funzionante. 😉

## Licenza

La skill è pubblicata come materiale operativo personale di Andrea Armeni. Se vuoi riutilizzarla o adattarla, verifica la licenza indicata nel file `SKILL.md` o contattami.
