Szybkie połączenie internetowe przez UDP. Wymyślone przez google, w celu obejścia TCP. Ponieważ implementacja QUIC jako surowe gniazdo wymagała by uprawnień _root_-a do korzystania z tego protokołu lub wszystkie systemy operacyjne musiałby by dodać go do listy protokołów, zaimplementowali QUIC tak aby wykorzystywał UDP, a co za tym idzie nie wymaga rozwiązania żadnego z wyżej wymienionych problemów.

Pierwotnie (2012): Quick UDP Internet Connections, aka TCP/2
Protokół połączeniowy ogólnego przeznaczenia
Warstwa transportowa
Google Chrome: ponad połowa połączeń z usługami Google
Lepsza współpraca z HTTP/2
Redukcja opóźnień
- HTTP over TLS (handshake TCP + handshake TLS)
- zintegrowane inicjowanie połączeń bezpiecznego
- szyfrowanie wybranych pakietów

#### Inne elementy QUIC
- unikalne identyfikatory połączeń -> możliwość kontynuacji transmisji po zmainie IP
- problem ograniczania ruchu opartego na UDP -> równoległe zestawienie połączeń QUIC i TCP + fallback
- inne zastosowania -> DNS-over-QUIC