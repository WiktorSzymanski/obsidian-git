### Google: SPDY
---
> Celem było przyśpieszenie HTTP głównie poprzez redukcję opóźnień oraz poprawę bezpieczeństwa. SPDY nie zastępuje HTTP, zmienia on sposób przesyłania żądań w sieci.
> #TODO 

#### Optymalizacje
- unikanie ponawiania wysyłania pól nagłówkowych
- kompresja nagłówków
- multipleksacja wielu żądań (można zacząć przesyłanie wielu żądań)
- priorytetiwanie
- "wypychanie" zasobów (_server push_)
- wskazywanie zasobów zależnych (_server hint_)
###### efekt: przyśpieszenie na poziomie 40%

#### Zasoby zależne
- Server Push
	- **wysyłanie zasobów zależnnych bez żądań**, w momencie gdy **klient nie potrzebuje tego zasobu** lub gdy **ma go już w swoim cache** może to powodować **zbędne żądania**.
	#TODO 

### Microsoft: Speed + Mobility
---
- Prace nad HTTP 2.0
- Propozycja Microsoftu
- Uwzględnianie kosztów związanych z:
	- energią
	- opłatami za przesył danych
- Kompresja, _server push_ a urządzenia mobilne #TODO 

### HTTP/2.0
---
- RFC 7540
- Wydany w maju 2015
##### Krytyka:
 - Bardzo krótki czas opracowania standardu
 - powielanie funkcji warstwy 4 (_flow control_ - dostosowanie prędkości do szybkości łącza tak aby rzadne urządzenie nie odrzuciło danych)
 - _de facto_ wymaga szyfrowania - czyli cały czas HTTPS
	 - w wielu zastosowaniach zbędne
	 - kłopotliwe zarządzanie certyfikatami
 - HTTP ponad SCTP - protokół warstwy 4, większość funkcji HTTP/2 nie było by potrzebnych podczas używania SCTP.

#### HTTP/2 zachowuje z HTTP/1.1
- metody
- kody błędów
- pola nagłówkowe
- adresy URI
##### Zmiana dotyczy sposobu upakowania wywołań (_framing_)
##### Multipleksacja:
> przeplot żądań i odpowiedzi w celu uniknęcia blokowania przez czasochłonne żądania (_head-of-line blocking_)

Kompresja nagłówków
Priorytety żądań
Eliminacja
#TODO

##### HTTP/1.1 pipelining
![[pipelining-1-1675947131.png]]

- Head-of-line blocking
- buggy proxy servers - wiele serwerów proxy nie było na to gotowych

#### HTTP/1.1 framing
#TODO 

#### Multipleksacja


#### HTTP/2 Push
- Nie jest to mechanizm powiadamiania klienta przez serwer
- Pojedyńcze żądanie + wiele odpowiedzi
- żądanie musi wystąpić
##### Push
- ramka PUSH_PROMISE załącza do normalnej odpowiedzi (na początku)
- #TODO 

#### Pojedyńcze łącze TCP
- HTTP/1.1 - 75% połączeń obsługuje 1 żądanie
- HTTP/2.0 - 25% połączeń obsługuje 1 żądanie
- TCP jest zoptymalizowany pod kątem długich, dużych transferów
- mniej połączeń TCP -> mniej zasobów (więcej zasobów może być pobrane jednym połączeniem TCP)
- dużo tańsze połączenie HTTPS, ponieważ każe żądanie TCP wiąże się z kosztem (_handshake_ połączenia i _handshake_ kodowania, stąd im więcej prześlemy jednym połączeniem TCP tym lepiej)

#### Flow control
- konfigurowany na poziomie protokołu HTTP
- ukierunkowany
- określany na #TODO 

#### HTTP a TCP
- TCP - niezawodny protokół połąćżeniowy (strumień)
- Retransmisja pakietu: pozostałe ramki (mimo że dotarły), pomimo multipleksacji muszą czekać: _head-of-line-blocking_
- Rozwiązanie: wykorzystanie UDP zamiast TCP
- Konieczność implementacji:
	- niezawodnego dostarczania pakietów
	- kotroli przepływu (_flow control_)

Tak powstał protokół [[QUIC]]

#### Kontrola przepływu (_flow control_)
- QUIC implementuje FC na poziomie aplikacji a nie jądra OS
- Oddzielny FC dla każdego strumienia
- Implementacja nowych algorytmów FC
- Możliwośc implementacji forward error correction (FEC)
	- nadmiarowy kod korygujący (error correcting code/ ECC)
	- eliminacja potrzeby retransmisji kosztem stałego narzutu
- Problem kostnienia protokołów (protocol ossification)
	- np. wdrożenie SCTP
- 2018: #BSR 

