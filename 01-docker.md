- `echo 'PS1="> "' > ~/.bashrc` — sovrascrive il file `.bashrc` impostando il prompt
  del terminale a `>` invece del default `user@DESKTOP:~$`.
  ⚠️ Usare `>` cancella tutto il contenuto precedente del file.
  Usare `>>` invece aggiunge senza cancellare.

  ## Docker - Concetti base

- **Docker Image** — è uno snapshot statico, un "modello" con tutto già installato
  (OS, pacchetti, dipendenze). Si crea una volta sola dal `Dockerfile`.

- **Docker Container** — è un'istanza in esecuzione di una Image. 
  Puoi avviare più container dalla stessa image.

- ⚠️ I package NON si installano ogni volta che entri nel container.
  Si installano UNA VOLTA SOLA nel `Dockerfile` quando costruisci la Image:
```dockerfile
  FROM python:3.11
  RUN pip install pandas sqlalchemy
```
  Poi lanci il container con `docker run` e trovi già tutto installato.

- Il flusso corretto è:
  1. Scrivi il `Dockerfile` con i package che ti servono
  2. `docker build` → crei la Image
  3. `docker run` → avvii il Container dalla Image

  - `docker run -it python:3.13.1-slim` — avvia un container con Python 3.13 
  in versione **slim** (leggera).
  
  - **slim** = image ridotta, contiene solo il minimo indispensabile per 
    far girare Python. Niente tool extra, pesa molto meno.
  - Ideale per produzione o quando vuoi container leggeri e veloci.
  
  | Versione | Dimensione | Uso |
  |----------|------------|-----|
  | `python:3.13` | ~1GB | sviluppo, ha tutto |
  | `python:3.13-slim` | ~150MB | produzione, essenziale |