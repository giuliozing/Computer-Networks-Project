# Trivia Quiz — C Client/Server

Project for the **Computer Networks** course (prof. Giuseppe Anastasi) — University of Pisa, A.Y. 2024/2025. Graded with full marks.

A multi-user trivia game built on TCP sockets: a multithreaded server handles several clients at once, each of which can register with a nickname, pick a topic among those available, and answer a series of questions, with score and leaderboard shared across all participants.

## Main features

- **Concurrency with threads**: the server spawns a POSIX thread for each connected client and a dedicated thread for the control panel, avoiding the overhead of context switches between processes.
- **Binary search trees**: users and leaderboards are kept in BSTs (ordered by nickname and by score/nickname respectively), for efficient ordered insertion, deletion and printing.
- **Server control panel**: a separate thread periodically (interval configurable) displays the list of topics, connected participants, the leaderboard for each topic and who has completed each topic; the server can be shut down cleanly by typing `q`.
- **Custom quiz format**: questions are organized in text files using a simple delimiter-based format (see below), parsed when the server starts. Adding new topics requires no code changes.
- **Minimal application protocol**: client and server exchange messages with a 4-byte header (length in network byte order) followed by the payload; no "message type" field is needed, since at every stage of the conversation both sides already know what to expect.
- **Signal handling**: `SIGPIPE` is explicitly handled on both the client and server side, to avoid abrupt process termination when the other end closes the connection, delegating error handling to the send functions.

## Project structure

```
.
├── server.c              # server entry point: socket setup, accept loop, thread startup
├── client.c              # client entry point: menu, user interaction
├── include/               # public headers for each module
│   ├── common.h           # shared utility functions (socket I/O, error handling)
│   ├── database.h         # data structures and functions for users/leaderboards (BST)
│   ├── dashboard.h         # server control panel
│   ├── game.h             # handling of a single client's game session
│   ├── params.h           # configuration constants (port, timeouts, max lengths, ...)
│   └── quiz.h             # data structures and parsing of quiz files
├── modules/               # server-side module implementations
│   ├── common.c
│   ├── database.c
│   ├── dashboard.c
│   ├── game.c
│   └── quiz.c
├── quiz/                  # quiz content (topics and questions)
│   ├── indice.quiz        # number of topics and their names
│   └── N.quiz             # questions and answers for topic N
├── Makefile
└── start.sh               # builds the project and launches a server and two test clients in separate terminals
```

## Building and running

Requires `gcc` and `make` on a Linux/POSIX system.

```bash
make          # build server and client
./server      # start the server (port and IP configurable in include/params.h)
./client 3000 # start a client, specifying the server's port
```

Alternatively, `start.sh` builds the project and automatically starts a server and two clients in separate terminal windows (requires `gnome-terminal`):

```bash
./start.sh
```

To remove build artifacts:

```bash
make clean
```

## `.quiz` file format

- `quiz/indice.quiz` contains, on the first line, the number of available topics, followed by one line per topic with its name.
- Each topic is defined in a file `N.quiz` (where `N` is the topic's position in the index, starting from 1), with one line per question.
- On each line, the question text is separated from the accepted answers by a `|` character; multiple valid answers to the same question are separated by `~`.

Example (`quiz/1.quiz`):

```
Chi ha scritto la "Divina Commedia"?|dante alighieri~dante
```

This format allows topics and questions to be added or edited simply by creating/editing text files, without recompiling the code.

## License

Distributed under the [MIT License](LICENSE).
