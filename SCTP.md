#SRC #Sem1 #PS #TODO 

### Stream Control Transmission Protocol
---
...
Mówi sie że UDP jest zawodny, jednak oznacza to tyle, że nie otrzymujemy potwierdzenia że pakiet został odebrany. UDP jest tak niezawodny jak połączenie którym jest wysyłany.

###### Terminologia protokołu:
- _połączenie_
- _asocjacja_
- _strumień_
### Cechy protokołu SCTP
---
- wykorzystuje komunikację strumieniową (dla pojedyńczych _asocjacji_) lub pakietową/datagramową (dla wielu _asocjacji_)
- zawiera mechanizmy zwiększające bezpieczeństwo i niezawodność komunikacji
- domyślnie zachowuje porządek wiadomości (ang. _ordered dilivery_) (FIFO), pozwala zmienić komunikację na niezachowującą (ang. _unordered dilivery_)
- umożliwia realizację połączeń komunikacji łączem pojedyńczym lub wieloma łączami (ang _multi-homing_). Pozwala to odbierać te same pakiety na każdnym z interfejsów sieciowych.
- ...

### Zestawienie oraz Zamykanie połączenia
---
... obrazek z nawiązywaniem połączenia
W SCTP zasoby dla gniazda sieciowego są dopiero po otrzymaniu `COOKIE-ECHO` z wcześniej wylosowanym przez serwer `StateCookie`. Stąd Four-way Handshake jest bezpieczniejszy od klasycznego Three Way handshake.

Protokół TCP, do zamknięcia połączenia, wykorzystuje procedurę wymiany dwóch/czterech komunikatów: `FIN` oraz `ACK`, co może prowadzić do połączeń "pół-otwartych".

Protokół SCTP, do zamykania połączenia
...
... obrazek TCP vs SCTP Connection Termination

### Komunikacja wieloma łączami (ang. _multi-homing_)
---
Bez żadnej konfiguracji SCTP domyślnie wykorzystuje wszystkie dostępne interfejsy komputera, ale możemy zkonfigurować których może. Pozwala to na niezawodność komunikacji. Gdy jedna przestanie działać SCTP przełączy się na drugą bez utraty żadnych danych. W przypadku gdy wykryje, że wcześniejszy interfejs działa poprawnie, bezstratnie może do niego powrócić. Dodatkowo można ustawić faworyzowany interfejs.
...
... obrazek 


### Komunikacja wieloma strumieniami (ang. _multi-streaming_)
---
W protokole SCTP, w ramach pojedyńczego połączenia możliwe jest utworzenie wielu niezależnych strumieni komunikacyjnych (identyfikowanych numerami).

Pozwala to na wysyłanie danym strumieniem pewnego rodzaju danych np. logów, video, audio. (Chyba) Gdy jeden ze strumieni przestanie działać, wysyłane przez niego pakiety rozłożą się na pozostałych.

...
... obrazek

Wymaga odpowieniego odpalenia gniazd, aby je obsłóżyć. 

### Gniazda sieciowe wielu asocjacji
---
Pojedyńcze gniazdo sieciowe protokołu SCTP pozwala na zestawienie połączenia z wieloma zdalnymi procesami jednocześnie - każde z tych połączeń nazywane jest _asocjacją_.

Z punktu widzenia TCP dla każdego nawiązanego połączenia jest potrzebne gniazdo. Dla przykładu dla 100 połączeń potrzebne jest 100 deskryptorów. Wymaga to multipleksacji wejścia/wyjścia aby nie blokować procesu. W SCTP na jednym deskryptorze odbieramy wszystkie pakiety i rozpoznajemy je po identyfikatorze. Jest po przydatne przy broadcast-cie.

### Protokół SCTP w systemach operacyjnych GNU/Linux
---
...

##### Przykład
- Serwer
``` bash
sctp_darn -H 0 -P 2500 -l
```
- Klient
``` bash
sctp_darn -H 0 -P 2600 -h 127.0.0.1 ...
```

Komunikacja TCP może być tunelowana protokołem SCTP.

W ramach narzędzi protokołu SCTP w systemach operacyjnych GNU/Linux dostępny jest program `withsctp(1)`, który pozwala na tego typu tunelowanie.

##### Przykład
``` bash
withsctp telnet HOST PORT
```

Do pisania aplikacji obsługujących SCTP można wykorzystać [[Gniazda Sieciowe Protokołu SCTP]].

### Pyrania i zadania
1. ...
2. ...
3. ...