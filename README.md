# Trivia Quiz — Client/Server in C

Progetto per il corso di **Reti di Calcolatori** (prof. Giuseppe Anastasi) — Università di Pisa, A.A. 2024/2025. Valutato con il massimo dei voti.

Un gioco di trivia multi-utente basato su socket TCP: un server multithread gestisce più client contemporaneamente, ciascuno dei quali può registrarsi con un nickname, scegliere un tema tra quelli disponibili e rispondere a una serie di domande, con punteggio e classifica condivisi tra tutti i partecipanti.

## Funzionalità principali

- **Concorrenza con thread**: il server crea un thread POSIX per ogni client connesso e un thread dedicato al pannello di controllo, evitando l'overhead dei cambi di contesto tra processi.
- **Alberi binari di ricerca**: utenti e classifiche sono mantenuti in BST (ordinati rispettivamente per nickname e per punteggio/nickname), per inserimento, cancellazione e stampa ordinata efficienti.
- **Pannello di controllo del server**: un thread separato mostra a intervalli regolari (configurabili) l'elenco dei temi, i partecipanti connessi, la classifica per tema e chi ha completato ciascun tema; il server può essere terminato in modo pulito digitando `q`.
- **Formato quiz personalizzato**: le domande sono organizzate in file di testo con un semplice formato a delimitatori (vedi sotto), parsati all'avvio del server. Aggiungere nuovi temi non richiede modifiche al codice.
- **Protocollo applicativo minimale**: client e server si scambiano messaggi con un header di 4 byte (lunghezza in network byte order) seguito dal payload; non serve un campo "tipo messaggio" perché, in ogni fase della conversazione, entrambe le parti sanno a priori cosa aspettarsi.
- **Gestione dei segnali**: `SIGPIPE` è gestito esplicitamente sia lato client che lato server, per evitare la terminazione brusca del processo quando l'altra estremità chiude la connessione, delegando la gestione dell'errore alle funzioni di invio.

## Struttura del progetto

```
.
├── server.c              # entry point del server: setup socket, accept loop, avvio thread
├── client.c              # entry point del client: menu, interazione con l'utente
├── include/               # header pubblici dei moduli
│   ├── common.h           # funzioni di utilità condivise (I/O su socket, gestione errori)
│   ├── database.h         # strutture dati e funzioni per utenti/classifiche (BST)
│   ├── dashboard.h        # pannello di controllo del server
│   ├── game.h             # gestione della partita di un singolo client
│   ├── params.h           # costanti di configurazione (porta, timeout, lunghezze massime, ...)
│   └── quiz.h             # strutture dati e parsing dei file di quiz
├── modules/               # implementazione dei moduli lato server
│   ├── common.c
│   ├── database.c
│   ├── dashboard.c
│   ├── game.c
│   └── quiz.c
├── quiz/                  # contenuti dei quiz (temi e domande)
│   ├── indice.quiz        # numero di temi e relativi nomi
│   └── N.quiz             # domande e risposte del tema N
├── Makefile
└── start.sh               # compila ed avvia un server e due client di test in terminali separati
```

## Compilazione ed esecuzione

Richiede `gcc` e `make` su un sistema Linux/POSIX.

```bash
make          # compila server e client
./server      # avvia il server (porta e IP configurabili in include/params.h)
./client 3000 # avvia un client, specificando la porta del server
```

In alternativa, lo script `start.sh` compila il progetto e avvia automaticamente un server e due client in finestre di terminale separate (richiede `gnome-terminal`):

```bash
./start.sh
```

Per pulire i file generati dalla compilazione:

```bash
make clean
```

## Formato dei file `.quiz`

- `quiz/indice.quiz` contiene, sulla prima riga, il numero di temi disponibili, seguito da una riga per ciascun tema con il relativo nome.
- Ogni tema è definito in un file `N.quiz` (dove `N` è la posizione del tema nell'indice, a partire da 1), con una riga per domanda.
- In ogni riga, il testo della domanda è separato dalle risposte accettate dal carattere `|`; più risposte valide per la stessa domanda sono separate dal carattere `~`.

Esempio (`quiz/1.quiz`):

```
Chi ha scritto la "Divina Commedia"?|dante alighieri~dante
```

Questo formato permette di aggiungere o modificare temi e domande semplicemente creando/editando file di testo, senza ricompilare il codice.

## Licenza

Distribuito con licenza [MIT](LICENSE).
