---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 61
---
# 61. Architektury serwerów sieciowych
---
> **Architektura serwera** określa, jak serwer **obsługuje wielu klientów jednocześnie**: ile procesów/wątków tworzy, jak czeka na wejście/wyjście i jak rozkłada pracę na rdzenie. Wybór wpływa na **skalowalność** (liczbę obsługiwanych połączeń), **wydajność**, złożoność i zużycie zasobów. Podstawą jest wybór **modelu I/O** – [[60 Obsługa operacji wejścia-wyjścia komunikacji sieciowej]].

## Schemat serwera TCP
```c
sfd = socket(PF_INET, SOCK_STREAM, 0);
setsockopt(sfd, SOL_SOCKET, SO_REUSEADDR, ...);
bind(sfd, ...);
listen(sfd, backlog);              // kolejka oczekujących połączeń
while (1) {
    cfd = accept(sfd, ...);        // nowe połączenie
    obsluz(cfd);                    // <-- tu różnią się architektury
}
```
Kod bazowy (klient/serwer TCP i UDP): [[Programowanie Sieciowe/Podstawowe Gniazda Sieciowe]]. Deskryptory: [[Programowanie Sieciowe/Deskryptor]], [[Programowanie Sieciowe - 01]].

## Serwer iteracyjny vs współbieżny
- **Iteracyjny** (_iterative_) – obsługuje **jedno połączenie naraz** (kolejne czekają w kolejce `listen`). Prosty, wystarczający dla krótkich, szybkich żądań (np. UDP echo, prosty DNS). Wada: długie żądanie blokuje pozostałych.
- **Współbieżny** (_concurrent_) – obsługuje **wiele połączeń jednocześnie**. Konieczny dla żądań z I/O (baza, sieć) lub trwałych połączeń. Realizowany procesami, wątkami lub zdarzeniami.

## Architektury współbieżne
### 1. Proces na połączenie (_process-per-connection_, fork)
```c
while (1) {
    cfd = accept(sfd, ...);
    if (fork() == 0) {              // proces potomny
        close(sfd);
        obsluz(cfd);
        exit(0);
    }
    close(cfd);                     // rodzic wraca do accept
    // obsługa SIGCHLD -> waitpid (unikanie zombie)
}
```
- Proces potomny **dziedziczy deskryptor** połączenia (klasyczny model UNIX – Apache prefork, inetd, dawny sendmail).
- ✔ pełna izolacja (awaria/wyciek pamięci jednego klienta nie psuje innych), bezpieczeństwo (osobne przestrzenie adresowe, można zrzucić uprawnienia), prostota kodu (kod sekwencyjny);
- ✘ **kosztowne**: tworzenie procesu, pamięć na proces, przełączanie kontekstu → praktyczny limit setek–tysięcy połączeń; współdzielenie stanu wymaga IPC.

### 2. Prefork (pula procesów)
Serwer tworzy **z góry pulę** procesów potomnych; każdy w pętli sam wykonuje `accept` na współdzielonym gnieździe nasłuchującym (jądro rozdziela połączenia; ochrona przed _thundering herd_ – blokada wokół `accept` lub `SO_REUSEPORT`).
- ✔ brak kosztu `fork` na połączenie, izolacja procesów, dynamiczne dostosowanie liczby procesów (Apache MPM prefork, PHP-FPM, Gunicorn, PostgreSQL);
- ✘ nadal pamięć na proces, ograniczona skalowalność.

### 3. Wątek na połączenie (_thread-per-connection_)
```c
while (1) {
    cfd = accept(sfd, ...);
    int *arg = malloc(sizeof(int)); *arg = cfd;
    pthread_create(&tid, NULL, obsluz_watek, arg);
    pthread_detach(tid);
}
```
- ✔ tańsze niż procesy (wspólna przestrzeń adresowa – łatwe współdzielenie stanu, mniej pamięci, szybsze tworzenie), naturalny kod sekwencyjny;
- ✘ **synchronizacja** wspólnych danych (blokady, wyścigi – [[09 Wielozadaniowość i synchronizacja zadań i wątków]]), koszt pamięci stosu na wątek i przełączania kontekstu przy tysiącach wątków, brak izolacji (awaria wątku może wywrócić proces).

### 4. Pula wątków (_thread pool_, prethreading)
Stała pula wątków roboczych + **kolejka zadań/połączeń**. Wątek akceptujący (lub wszystkie wątki) wkłada połączenia do kolejki, robocze je pobierają (producent-konsument – [[37 Monitory w C Sharp i Java]]).
- ✔ ograniczona, kontrolowana liczba wątków (dopasowana do rdzeni), brak kosztu tworzenia na połączenie, unikanie przeciążenia;
- ✘ długie operacje blokujące zajmują wątek (wyczerpanie puli), potrzebne dobranie rozmiaru puli.

### 5. Zdarzeniowa (_event-driven_) – wzorzec Reactor
**Jeden wątek** z **pętlą zdarzeń** i **multipleksacją** (`epoll`/`kqueue`), gniazda **nieblokujące** – [[60 Obsługa operacji wejścia-wyjścia komunikacji sieciowej]].
```c
int ep = epoll_create1(0);
epoll_ctl(ep, EPOLL_CTL_ADD, sfd, &(struct epoll_event){.events=EPOLLIN, .data.fd=sfd});
while (1) {
    int n = epoll_wait(ep, events, MAX, -1);
    for (int i = 0; i < n; i++) {
        if (events[i].data.fd == sfd) {
            while ((cfd = accept4(sfd, ..., SOCK_NONBLOCK)) >= 0)
                epoll_ctl(ep, EPOLL_CTL_ADD, cfd, ...);   // dodaj nowe połączenie
        } else {
            obsluz_nieblokujaco(events[i].data.fd);        // czytaj/pisz bez blokowania
        }
    }
}
```
- **Reactor** (Schmidt): demultiplekser zdarzeń (epoll) → **dispatcher** wywołuje **handlery** zarejestrowane dla zdarzeń (gotowość do odczytu/zapisu); handler wykonuje krótką, nieblokującą pracę i rejestruje kolejne zainteresowanie.
- ✔ obsługa **dziesiątek/setek tysięcy** połączeń małą liczbą wątków, mało pamięci i przełączeń kontekstu (**rozwiązanie C10K**), świetne dla dużej liczby wolnych/trwałych połączeń (WebSocket, long polling, proxy);
- ✘ **nie wolno blokować pętli** (blokujące API i długie obliczenia CPU przenosi się do puli wątków), kod z callbackami/maszyną stanów trudniejszy w utrzymaniu, jeden wątek nie wykorzysta wielu rdzeni.
- Przykłady: **nginx**, **HAProxy**, **Redis** (jednowątkowy event loop), **Node.js** (libuv), lighttpd, Netty.

### 6. Proactor – asynchroniczne I/O
Handler wywoływany po **zakończeniu** operacji I/O (a nie gotowości): POSIX AIO, **io_uring** (Linux), **IOCP** (Windows). Aplikacja zleca operacje, jądro je wykonuje i powiadamia. Wydajny, ale trudniejszy; io_uring zyskuje popularność.

### 7. Architektury hybrydowe i wielordzeniowe
- **wiele procesów/wątków, każdy z własną pętlą zdarzeń** (epoll) – wykorzystanie wszystkich rdzeni:
  - **nginx**: proces master + wielu **workerów**, każdy z pętlą epoll; `SO_REUSEPORT` rozdziela nowe połączenia między workery w jądrze (bez rywalizacji o accept),
  - **SEDA** (_Staged Event-Driven Architecture_, Welsh) – przetwarzanie podzielone na **etapy** połączone kolejkami, każdy z własną pulą wątków i kontrolą przeciążenia,
  - model **worker-per-core** (Seastar, DPDK) – po jednym wątku na rdzeń, dane przypięte do rdzenia (współdzielenie przez komunikaty, bez blokad),
- **pula procesów + pula wątków** (Apache MPM worker/event: procesy, w każdym wątki; MPM event – wątek dedykowany epoll do połączeń keep-alive),
- podział na wątki I/O (akceptujące, epoll) i wątki robocze (logika/CPU).

## Wybór architektury
| Architektura | Skalowalność | Izolacja | Wykorzystanie rdzeni | Złożoność | Przykłady |
|---|---|---|---|---|---|
| iteracyjny | 1 klient | – | 1 rdzeń | najniższa | proste UDP, DNS |
| proces/połączenie | niska–średnia | **pełna** | wiele (proces = rdzeń) | niska | inetd, CGI |
| prefork | średnia | pełna | tak | niska | Apache prefork, PHP-FPM |
| wątek/połączenie | średnia | brak | tak | średnia (synchronizacja) | Apache worker, serwery Java (Tomcat blocking) |
| pula wątków | średnia–wysoka | brak | tak | średnia | serwery aplikacji, Tomcat |
| zdarzeniowa (Reactor) | **bardzo wysoka** | brak | **1 rdzeń** (bez rozszerzeń) | wysoka | nginx (worker), Redis, Node.js |
| zdarzeniowa × N rdzeni | **najwyższa** | częściowa | **wszystkie** | wysoka | nginx, HAProxy, Netty, Seastar |
| proactor (io_uring/IOCP) | najwyższa | brak | wszystkie | wysoka | serwery Windows, nowe serwery Linux |

**Wskazówki**:
- mało długich, obliczeniowych żądań → procesy/wątki (prostota, izolacja, wykorzystanie rdzeni),
- dużo połączeń z przewagą I/O (web, proxy, komunikatory, WebSocket) → **zdarzeniowa** na wielu workerach,
- bezpieczeństwo/izolacja krytyczna → procesy z ograniczeniem uprawnień,
- współczesne serwery łączą podejścia (event loop na rdzeń + pula wątków na blokujące zadania).

## Powiązane zagadnienia
- **backlog** (`listen`), `SYN cookies` (ochrona przed SYN flood – [[TCP]]),
- keep-alive, timeouty, limity połączeń, graceful shutdown,
- **load balancing** przed farmą serwerów (L4/L7, DNS), bezstanowość dla skalowania poziomego – [[47 Architektury systemów rozproszonych dużej skali]],
- asynchroniczne serwery aplikacyjne i frameworki – [[16 Asynchroniczna implementacja serwerów usług]],
- WebSocket i długotrwałe połączenia wymagają architektury zdarzeniowej – [[15 Asynchroniczna komunikacja HTTP i WebSocket]].

## Zobacz też
- [[60 Obsługa operacji wejścia-wyjścia komunikacji sieciowej]]
- [[Programowanie Sieciowe/SCTP]] (multi-streaming/multi-homing w jednym gnieździe)
