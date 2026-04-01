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

 ### Vedere tutti i container
```bash
docker ps -a
```
- Senza `-a` mostra solo i container attivi
- Le colonne principali: `CONTAINER ID`, `IMAGE`, `STATUS`, `NAMES`

## Bash - Comandi base

### Creare cartelle e file
```bash
mkdir test                                # crea la directory "test" nella cartella corrente
cd test                                   # entra nella directory "test" (cd = change directory)
touch file1.txt file2.txt file3.txt       # crea tre file vuoti
echo "Hello from host" > file1.txt        # scrive la stringa dentro file1.txt
cd ..                                     # torna alla directory padre
```
- `touch` crea file vuoti; se il file esiste già, aggiorna solo la data di modifica
- `>` è la **redirezione dell'output**: invece di stampare a schermo, manda il risultato nel file (sovrascrive)
- `>>` fa la stessa cosa ma in **append** (aggiunge in fondo senza sovrascrivere)

---

## Docker - Volumi

I volumi collegano una cartella dell'**host** a una cartella dentro il **container**.
Senza volumi, tutto ciò che crei dentro un container viene perso quando lo fermi.

### Montare un volume
```bash
docker run -it --entrypoint=bash -v $(pwd)/test:/app/test python:3.13.1-slim
```
- `-it` — modalità interattiva con terminale
- `--entrypoint=bash` — sovrascrive il comando di default dell'immagine (che sarebbe l'interprete Python `>>>`) e apre una shell bash
- `-v <path_host>:<path_container>` — monta il volume, i due percorsi separati da `:`
- `$(pwd)` — restituisce la directory corrente sull'host
- I file rimangono sull'host anche dopo che il container viene eliminato
```
HOST (Codespaces)            CONTAINER
$(pwd)/test/          →      /app/test/
  ├── file1.txt                ├── file1.txt  ("Hello from host")
  ├── file2.txt                ├── file2.txt
  └── file3.txt                └── file3.txt
```

> Le modifiche fatte dentro il container si riflettono sull'host e viceversa, in tempo reale.

### Verificare il volume dal container
```bash
ls /app/test              # vedi i file montati
cat /app/test/file1.txt   # stampa "Hello from host"
```

### Navigare dentro il container ed eseguire uno script
```bash
cd app/         # entra in /app
cd test/        # entra in /app/test (cartella montata dal volume)
python script.py  # esegue lo script nel container

cd ..           # torna su di una cartella (da /app/test → /app, da /app → /)
cd /            # torna direttamente alla root del container da qualsiasi punto
```
- il prompt `root@75a43b865b22:/app/test#` conferma che sei dentro il container
  - `root` = utente con tutti i permessi
  - `75a43b865b22` = ID del container (stesso che vedi con `docker ps`)
  - `/app/test` = directory corrente nel container
  - `#` = sei root (`$` per utenti normali)
- si può accorciare con: `cd app/test && python script.py`

```bash
exit            # esci dalla shell bash del container e torni sull'host
                # il container passa a STATUS: Exited (visibile con docker ps -a)
                # i file nel volume (test/) rimangono intatti sull'host
```

### Script Python per esplorare i file nel container
```python
from pathlib import Path

current_dir = Path.cwd()
current_file = Path(__file__).name

print(f"Files in {current_dir}:")

for filepath in current_dir.iterdir():
    if filepath.name == current_file:
        continue

    print(f"  - {filepath.name}")

    if filepath.is_file():
        content = filepath.read_text(encoding='utf-8')
        print(f"    Content: {content}")
```
- `Path.cwd()` — restituisce la directory corrente dentro il container
- `iterdir()` — itera su tutti i file e cartelle nella directory
- Salta se stesso (`current_file`) per non stampare il proprio contenuto
- Legge e stampa il contenuto di ogni file trovato