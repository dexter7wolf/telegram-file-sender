---
name: telegram-file-sender
description: Invia file arbitrari a Telegram copiandoli prima nella cache documenti consentita. Usa quando devi inviare file locali (PDF, DOCX, XLSX, ZIP, immagini) che non si trovano nelle cartelle di cache di Hermes.
version: 1.0.0
author: Andrea Armeni
license: Proprietary
platforms: [linux]
metadata:
  hermes:
    tags: [telegram, file-sending, media-delivery, workaround]
---

# Telegram File Sender

## Contesto
Hermes media delivery funziona SOLO per file in cartelle consentite:
- `~/.hermes/cache/images/`
- `~/.hermes/cache/documents/`
- `~/.hermes/cache/screenshots/`
- `~/.hermes/image_cache/`
- `~/.hermes/document_cache/`

File in altre cartelle (es. `_inbox/raw`, progetti, download) vengono **scartati silenziosamente** dal gateway.

## Procedura

### 1. Copiare i file nella cache documenti (nomi puliti)
```bash
mkdir -p ~/.hermes/cache/documents/telegram-sender
cp /path/to/file.pdf ~/.hermes/cache/documents/telegram-sender/
```

**Regole nomi file:**
- Niente spazi (sostituisci con `-`)
- Niente caratteri speciali
- Massimo 50 caratteri
- Estensione originale conservata

### 2. Inviare documenti/PDF/ZIP/XLSX con Telegram API (curl)
```bash
BOT_TOKEN=$(grep TELEGRAM_BOT_TOKEN ~/.hermes/.env | cut -d= -f2)
CHAT_ID=<TELEGRAM_CHAT_ID>

curl -F "document=@/percorso/file.pdf" \
  "https://api.telegram.org/bot${BOT_TOKEN}/sendDocument" \
  -F "chat_id=${CHAT_ID}" \
  -F "caption=nomefile.pdf"
```

**Limite:** 1 file per chiamata curl (Telegram API).

### 3. Inviare immagini con MEDIA: (funziona)
```
MEDIA:${HOME}/.hermes/cache/documents/telegram-sender/foto.png
```

### 4. Pulizia (opzionale)
```bash
rm -rf ~/.hermes/cache/documents/telegram-sender/*
```

## Output atteso
- Messaggio Telegram con allegati nativi (PDF, documenti, immagini)
- Nessun errore silenzioso
- File consegnati come attachment, non come link

## Errori comuni
- "File non inviato" → il file non era nella cache consentita
- "Silent drop" → il gateway ha scartato il file senza avvisare
- Soluzione: verificare che il path sia sotto `~/.hermes/cache/`
