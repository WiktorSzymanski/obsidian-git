---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 60
---
# 60. Obsługa operacji wejścia/wyjścia komunikacji sieciowej
---
> Serwer obsługujący wiele połączeń musi zdecydować, **jak czekać na gotowość** wielu deskryptorów gniazd do odczytu/zapisu, nie blokując się na jednym i nie marnując procesora. **Modele wejścia/wyjścia** (klasyfikacja Stevensa, „UNIX Network Programming”) różnią się tym, kiedy i jak operacja I/O blokuje proces oraz kto wykonuje kopiowanie danych.

## Dwie fazy operacji odczytu
Operacja `read`/`recv` na gnieździe składa się z dwóch faz:
1. **oczekiwanie na gotowość danych** – dane muszą nadejść i trafić do bufora jądra,
2. **kopiowanie danych** z bufora jądra do bufora aplikacji.

Modele I/O różnią się tym, którą fazę i jak blokują.

## Modele wejścia/wyjścia (Stevens)
### 1. Blokujące I/O (_blocking_)
Domyślne. `recvfrom` **blokuje** proces przez **obie** fazy (aż dane nadejdą i zostaną skopiowane).
- ✔ proste; ✘ jeden deskryptor na proces/wątek → do obsługi wielu połączeń potrzeba wielu procesów/wątków ([[61 Architektury serwerów sieciowych]]).

### 2. Nieblokujące I/O (_nonblocking_)
Deskryptor w trybie `O_NONBLOCK` (`fcntl(fd, F_SETFL, O_NONBLOCK)` lub `FIOASYNC`/`FIONBIO` przez `ioctl`). `recvfrom` zwraca natychmiast: dane albo błąd **`EAGAIN`/`EWOULDBLOCK`**, gdy brak danych.
- Aplikacja musi **odpytywać** (_polling_ – busy wait) → marnuje procesor; rzadko używane samodzielnie, ale konieczne dla modelu zdarzeniowego (żeby `accept`/`read`/`write` nie zablokowały pętli).

### 3. Multipleksacja I/O (_I/O multiplexing_) – `select`/`poll`/`epoll`
Proces blokuje się na **jednej** funkcji (`select`), która monitoruje **wiele deskryptorów** i zwraca, gdy **którykolwiek** jest gotowy; potem `recvfrom` na gotowym deskryptorze nie zablokuje.
- Faza 1 (oczekiwanie) obsłużona dla wielu deskryptorów naraz jednym wywołaniem; faza 2 (kopiowanie) blokuje krótko.
- **Podstawa serwerów zdarzeniowych** (jednowątkowa obsługa tysięcy połączeń – [[16 Asynchroniczna implementacja serwerów usług]]).

### 4. I/O sterowane sygnałem (_signal-driven_, `SIGIO`)
Aplikacja włącza `SIGIO` (`fcntl` z `F_SETOWN`, `O_ASYNC`); jądro **wysyła sygnał**, gdy deskryptor jest gotowy; handler wykonuje `recvfrom`.
- Nie blokuje w fazie 1; rzadko używane dla TCP (sygnał nie mówi, który deskryptor i jaki typ gotowości), lepsze dla UDP i jednego deskryptora.

### 5. Asynchroniczne I/O (_asynchronous_, POSIX AIO / io_uring)
Aplikacja zleca operację (`aio_read`) z buforem; jądro wykonuje **obie fazy** (oczekiwanie **i** kopiowanie) i powiadamia po **zakończeniu** (sygnał, callback, zdarzenie).
- Aplikacja w ogóle się nie blokuje; różnica względem multipleksacji: tam powiadomienie o **gotowości** (potem trzeba samemu czytać), tu o **zakończeniu** całej operacji.
- POSIX AIO w Linuksie długo słabo wspierane (implementacja w glibc przez wątki); **io_uring** (2019) – wydajne, prawdziwie asynchroniczne I/O przez współdzielone kolejki (submission/completion ring), także dla plików, `accept`, `send`.

### Porównanie
| Model | Faza 1 (oczekiwanie) | Faza 2 (kopiowanie) | Wiele deskryptorów |
|---|---|---|---|
| blokujące | blokuje | blokuje | nie (wątek/proces na deskryptor) |
| nieblokujące | odpytywanie | blokuje | tak (busy wait – nieefektywne) |
| multipleksacja | blokuje na `select` (wiele fd) | blokuje | **tak** |
| sterowane sygnałem | sygnał | blokuje | ograniczone |
| asynchroniczne | nie blokuje | **nie blokuje** (jądro) | tak |

**synchroniczne** = 1–4 (faza 2 kopiowania blokuje proces); **asynchroniczne** = 5.

## Mechanizmy multipleksacji w Linuksie
### `select(2)`
```c
fd_set rfds; FD_ZERO(&rfds); FD_SET(sfd, &rfds);
struct timeval tv = { .tv_sec = 5 };
int n = select(maxfd + 1, &rfds, NULL, NULL, &tv);   // zbiory read/write/except + timeout
if (n > 0 && FD_ISSET(sfd, &rfds)) { /* sfd gotowe do odczytu */ }
```
- monitoruje trzy zbiory (odczyt, zapis, wyjątki) + timeout,
- **wady**: limit **`FD_SETSIZE`** (zwykle 1024), zbiory **modyfikowane** przy każdym wywołaniu (trzeba odbudowywać), jądro i aplikacja **skanują wszystkie** deskryptory → **O(n)** na wywołanie, kopiowanie zbiorów.

### `poll(2)`
```c
struct pollfd fds[N];
fds[0].fd = sfd; fds[0].events = POLLIN;
int n = poll(fds, N, timeout_ms);
if (fds[0].revents & POLLIN) { /* gotowe */ }
```
- tablica `pollfd` (fd, events, revents) – **brak limitu FD_SETSIZE**, rozdzielone events/revents (nie trzeba odbudowywać),
- nadal **O(n)** – jądro przegląda całą tablicę, kopiowanie tablicy przy każdym wywołaniu.

### `epoll(7)` (Linux)
```c
int ep = epoll_create1(0);
struct epoll_event ev = { .events = EPOLLIN, .data.fd = sfd };
epoll_ctl(ep, EPOLL_CTL_ADD, sfd, &ev);          // rejestracja raz
struct epoll_event events[MAX];
int n = epoll_wait(ep, events, MAX, timeout_ms);  // zwraca TYLKO gotowe
for (int i = 0; i < n; i++) { /* events[i].data.fd gotowe */ }
```
- deskryptory rejestrowane **raz** (`epoll_ctl`), jądro utrzymuje zainteresowanie i **listę gotowych**,
- `epoll_wait` zwraca **tylko gotowe** deskryptory → **O(liczby gotowych)**, skalowanie do **setek tysięcy** połączeń (rozwiązanie **C10K**),
- **tryby wyzwalania**:
  - **level-triggered (LT)** – domyślny, jak `poll`: zgłasza gotowość, **dopóki** są dane (bezpieczniejszy),
  - **edge-triggered (ET)** – zgłasza **tylko przy zmianie** (nadejściu nowych danych); wymaga czytania **do `EAGAIN`** w pętli i deskryptorów nieblokujących; mniej wywołań, wydajniejszy.
- odpowiedniki: **kqueue** (BSD, macOS), **IOCP** (_I/O Completion Ports_ – Windows, model completion jak AIO), `/dev/poll`, event ports (Solaris).

### Porównanie select/poll/epoll
| | select | poll | epoll |
|---|---|---|---|
| Limit deskryptorów | FD_SETSIZE (~1024) | brak | brak |
| Złożoność | O(n) | O(n) | O(gotowych) |
| Przekazywanie fd | zbiory za każdym razem | tablica za każdym razem | rejestracja raz |
| Przenośność | POSIX (wszędzie) | POSIX | Linux |
| Tryby | LT | LT | LT / ET |

## Powiązane funkcje I/O
- odczyt/zapis: `read`/`write`, `recv`/`send`, `recvfrom`/`sendto` (z adresem – UDP, surowe), `recvmsg`/`sendmsg` (dane pomocnicze, wiele buforów – _scatter/gather_ przez `iovec`), `readv`/`writev`,
- `sendfile(2)` – kopiowanie plik → gniazdo **w jądrze** (bez przechodzenia przez przestrzeń użytkownika – _zero-copy_), `splice`, `MSG_ZEROCOPY`,
- opcje gniazd: `SO_RCVBUF`/`SO_SNDBUF` (bufory), `SO_RCVTIMEO`/`SO_SNDTIMEO` (timeouty), `TCP_NODELAY` (wyłączenie algorytmu Nagle'a – małe pakiety bez opóźnienia), `SO_REUSEADDR`/`SO_REUSEPORT` (wiele gniazd na porcie – równoważenie w jądrze), `TCP_CORK`, keepalive.

## Zastosowanie w architekturach serwerów
Model I/O determinuje architekturę serwera:
- blokujące + proces/wątek na klienta,
- multipleksacja (`epoll`) → pętla zdarzeń (Reactor), pule wątków z epoll,
- asynchroniczne (io_uring, IOCP) → wzorzec Proactor.
Szczegóły: [[61 Architektury serwerów sieciowych]], [[16 Asynchroniczna implementacja serwerów usług]].

## Zobacz też
- [[Programowanie Sieciowe/Podstawowe Gniazda Sieciowe]], [[Programowanie Sieciowe/Gniazda Sieciowe Protokołu SCTP]] (multipleksacja połączeń w jednym gnieździe)
