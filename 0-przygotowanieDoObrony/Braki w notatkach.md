---
tags:
  - obrona
source: "[[egz_dypl_SRC_2024.pdf]]"
---
# Braki w notatkach

> [!success] Braki uzupełnione
> Dla **wszystkich 61 zagadnień** powstały notatki (foldery `01`–`11`), które pokrywają wypunktowane niżej braki. Aktualny indeks: [[Mapa zagadnień]]. Ta lista pozostaje jako **checklista zakresu** – każdy punkt powinien być omówiony w odpowiedniej notatce głównej; służy do samokontroli, czego szukać przy powtórce. Punkty odnoszące się do `#TODO` w oryginalnych notatkach vaultu (np. puste sekcje, brakujące obrazki) nadal warto uzupełnić **w tamtych** notatkach, jeśli chcesz mieć wszystko w jednym miejscu.

Lista rzeczy do omówienia (pierwotnie: braki przed utworzeniem notatek). Lokalizacje: [[Mapa zagadnień]].

- **🔴 Brak całkowity** – nie było notatki (utworzono nową notatkę główną).
- **🟡 Brak częściowy** – notatka istniała, ale niepełna (utworzono notatkę-uzupełnienie).

## Priorytety

Zagadnienia bez żadnego materiału (zaczynać od nich):
**4, 5, 6** (RSO) · **13, 14** (REST/ROA) · **20** (rozproszone systemy plików) · **26, 28, 29** (BSR web) · **30, 31, 32** (SNMP/MIB) · **38, 39, 40, 43, 44, 45** (MBP) · **48, 49, 50, 53, 54** (SRDS) · **60, 61** (PS)

Całe przedmioty praktycznie nieobecne w notatkach:
- **Rozproszone systemy operacyjne** (1–6) – są tylko rysunki z ćwiczeń.
- **Metody bezpiecznego programowania** (37–46) – poza [[Linearizability]] nie ma notatek z wykładu.
- **Zarządzanie systemami rozproszonymi** (30–32) – notatki są z DevOps, nie z zarządzania siecią.

---

## Rozproszone systemy operacyjne

### 1. Komunikacja grupowa 🔴
- [ ] Definicje: grupa procesów, grupa otwarta/zamknięta, statyczna/dynamiczna, członkostwo (*group membership*), widok (*view*)
- [ ] Rozgłaszanie niezawodne (*reliable broadcast*): własności (ważność, zgodność, integralność)
- [ ] Porządki dostarczania: FIFO, przyczynowy, totalny (atomowy) i ich kombinacje – **definicje RFB, RCB, RTB** (skróty są tylko w [[broadcast_zad_dom.excalidraw]])
- [ ] Algorytmy: rozgłaszanie FIFO (numery sekwencyjne), przyczynowe (zegary wektorowe), totalne (sekwencer, ISIS / uzgadnianie priorytetów)
- [ ] Synchronizacja widoków (*virtual synchrony*)
- [ ] Uzupełnić [[Algorytmy Rozproszone/Zgodne Rozgłaszanie Niezawodne FIFO]] (jedno zdanie) i [[Algorytmy Rozproszone/Gossiping]] (sekcje „Problem”, „Algorytmy rozpraszania” puste; niedokończony model SIR)
- [ ] Zegary logiczne Lamporta i wektorowe (podstawa porządku przyczynowego) – brak w całym vaultcie

### 2. Danocentryczne modele spójności 🟡
- [ ] [[Algorytmy Rozproszone/Model Spójności]] – dopisać treść definicji historii, uszeregowania legalnego, spójności atomowej (teraz „...”)
- [ ] Spójność sekwencyjna – formalna definicja (jest jedno zdanie)
- [ ] **Spójność przyczynowa** – definicja + relacja przyczynowości operacji
- [ ] **Spójność procesorowa (PRAM / FIFO)** – definicja (jest tylko w zadaniach w [[modele_spojnosci_zad_dom.excalidraw]])
- [ ] **Koherencja** (spójność „słaba” / *eventual*?) – brak jakiejkolwiek wzmianki; ustalić definicję z wykładu
- [ ] Hierarchia modeli (atomowa ⊂ sekwencyjna ⊂ przyczynowa ⊂ PRAM) z przykładem historii
- [ ] Opisać rozwiązania z rysunków [[AlgorytmyRozproszoneCwiczenia.excalidraw]], [[modele_spojnosci_zad_dom.excalidraw]] (teraz tylko zapis historii)

### 3. Modele spójności zorientowane na klienta 🔴
- [ ] Założenia modelu (klient mobilny, przełączanie serwerów, zbiory zapisów WS / odczytów RS)
- [ ] **RYW** – Read Your Writes
- [ ] **MR** – Monotonic Reads
- [ ] **MW** – Monotonic Writes
- [ ] **WFR** – Writes Follow Reads
- [ ] Przykłady naruszenia każdego modelu, implementacja (wektory wersji / znaczniki sesji)

### 4. Algorytmy wzajemnego wykluczania 🔴
- [ ] Wymagania: bezpieczeństwo, żywotność, uczciwość; miary (liczba komunikatów, opóźnienie synchronizacji)
- [ ] **Lamport** (kolejka żądań ze znacznikami czasowymi, 3(N−1) komunikatów)
- [ ] **Ricart-Agrawala** (REPLY z odroczeniem, 2(N−1))
- [ ] **Maekawa** – kworum √N, INQUIRE/FAILED/RELINQUISH, zakleszczenie (opisać rysunek [[Drawing 2024-06-19 16.38.35.excalidraw]])
- [ ] **Suzuki-Kasami** (token, tablice RN i LN)
- [ ] **Raymond** (token w drzewie, zmienna HOLDER)
- [ ] Tabela porównawcza złożoności komunikacyjnej
- [ ] Opisać odporny na awarie algorytm tokenu z [[Drawing 2024-12-11 08.09.32.excalidraw]] (są tylko pytania)

### 5. Algorytmy elekcji 🔴
- [ ] Definicja problemu i wymagania (bezpieczeństwo, żywotność)
- [ ] Algorytm tyrana (**Bully**, Garcia-Molina)
- [ ] Algorytmy pierścieniowe: **Chang-Roberts**, **Hirschberg-Sinclair**, LeLann
- [ ] Elekcja w drzewach / grafach dowolnych (opcjonalnie: przez fale)
- [ ] Złożoność komunikacyjna, porównanie
- [ ] Związek z wykrywaniem awarii i konsensusem (lider w Paxos/Raft)

### 6. Zakleszczenie w systemach rozproszonych 🔴
- [ ] Graf zależności (WFG), warunki zakleszczenia
- [ ] **Modele zakleszczeń**: jednozasobowy, AND, OR, AND-OR, p-z-q (k-z-n)
- [ ] Strategie: zapobieganie, unikanie, wykrywanie; fantomowe zakleszczenia
- [ ] **Chandy-Misra-Haas** – wersja dla modelu AND (komunikaty *probe*) i OR (zapytania/odpowiedzi, przetwarzanie dyfuzyjne)
- [ ] **Bracha-Toueg** – wykrywanie zakleszczenia w modelu p-z-q (NOTIFY/GRANT/DONE/ACK)
  - ⚠️ [[Systemy Wysokiej Niezawodności/Algorytm Bracha-Touega]] opisuje **algorytm konsensusu**, to inny algorytm tych samych autorów

---

## Narzędzia przetwarzania rozproszonego

### 7. Aspekty projektowe 🟡
- [ ] [[Narzędzia Przetwarzania Rozproszonego/Podstawowe Własności Systemu Rozproszonego]] – rozszerzalność (`#TODO`)
- [ ] Rodzaje przezroczystości – linki w [[Narzędzia Przetwarzania Rozproszonego/Przezroczystość]] prowadzą do nieistniejących notatek (relokacji, współbieżności, ukrywania awarii); [[Narzędzia Przetwarzania Rozproszonego/Przezroczystość Migracji]] pusta. Brakuje też: dostępu, położenia, replikacji, trwałości
- [ ] Heterogeniczność, bezpieczeństwo, obsługa awarii, współbieżność jako aspekty projektowe – zebrać w jednym miejscu
- [ ] Organizacja oprogramowania (middleware) – [[Zarządzanie Systemami Rozproszonymi/System Rozproszony]] ma `#TODO obrazki`

### 8. Podejścia do budowy systemów rozproszonych 🟡
- [ ] **Charakterystyka porównawcza** podejść (komunikaty vs RPC vs obiekty zdalne vs pamięć wspólna) – brak tabeli
- [ ] Model systemu dla każdego podejścia – `#TODO img z prezki` w [[Narzędzia Przetwarzania Rozproszonego/Zdalne Wywoływanie Procedur]]
- [ ] [[Narzędzia Przetwarzania Rozproszonego/Synchroniczność komunikacji]] – komunikacja asynchroniczna (`#TODO`)
- [ ] Semantyki RPC – dopisać „dokładnie raz” i „może” (maybe) w [[Narzędzia Przetwarzania Rozproszonego/Gwarancja wykonania (semantyka błędu)]]
- [ ] Podejście obiektowe (RMI/CORBA) – [[Narzędzia Przetwarzania Rozproszonego/Istota Podejścia Obiektowego]], [[Narzędzia Przetwarzania Rozproszonego/Wywoływanie metod zdalnych]] puste
- [ ] Middleware komunikatów (MPI, ZeroMQ – wzorce req/rep, pub/sub, push/pull) – [[Narzędzia Przetwarzania Rozproszonego/ZMQ]] to stub
- [ ] Ada: model rendezvous – sekcja „Współbieżność” w [[Narzędzia Przetwarzania Rozproszonego/ADA-95]] pusta

### 9. Wielozadaniowość i synchronizacja 🟡
- [ ] Zmienne warunkowe (`pthread_cond_wait/signal`) – [[Narzędzia Przetwarzania Rozproszonego/Zmienne Warunkowe]] pusta
- [ ] Ada: zadania (*task*), rendezvous (`entry`/`accept`/`select`), obiekty chronione (*protected*)
- [ ] Porównanie mechanizmów synchronizacji w narzędziach (pthreads vs Ada vs Java vs MPI)
- [ ] [[Wątek]] – współbieżność na 1 i wielu procesorach, concurrent vs parallel (`#TODO`)

---

## Technologie internetowe w przetwarzaniu rozproszonym

### 10. HTML5 API 🔴
- [ ] **Web Workers** – dedykowane/współdzielone, `postMessage`, brak dostępu do DOM
- [ ] **Web Storage** – `localStorage` vs `sessionStorage`, API, limity, zdarzenie `storage` ([[Technologie internetowe w przetwarzaniu rozproszonym/Web Storage]] to stub); IndexedDB
- [ ] **Service Worker** – cykl życia (install/activate/fetch), cache offline, wymóg HTTPS
- [ ] **Web Push** – Push API + Notifications API, VAPID, push service

### 11. SOA 🟢
- (brak istotnych braków)

### 12. SOAP i WSDL 🟢
- [ ] Drobne: sekcje „SOAP Body” i „SOAP Response” w [[Technologie internetowe w przetwarzaniu rozproszonym/SOAP Messages]] są puste
- [ ] Styl RPC/Document, literal/encoded (jest tylko wzmianka w [[Technologie internetowe w przetwarzaniu rozproszonym/WS-I]])

### 13. REST 🔴
- [ ] Ograniczenia architektury REST (Fielding): klient-serwer, bezstanowość, cache, jednolity interfejs, warstwy, code-on-demand
- [ ] Modelowanie usług: identyfikacja zasobów, kolekcje vs elementy, relacje
- [ ] Metody HTTP: semantyka GET/POST/PUT/PATCH/DELETE/HEAD/OPTIONS, **bezpieczeństwo i idempotentność**
- [ ] Rola URI – nazewnictwo, hierarchia, parametry
- [ ] Reprezentacje zasobów – negocjacja treści (`Accept`, `Content-Type`), JSON/XML
- [ ] Kody statusu, nagłówki (`Location`, `ETag`)
- [ ] [[Technologie internetowe w przetwarzaniu rozproszonym/REST]] – obecnie tylko skrót

### 14. ROA, HATEOAS, niezawodność HTTP 🔴
- [ ] ROA (Richardson & Ruby) – adresowalność, bezstanowość, połączalność, jednolity interfejs
- [ ] RPC a zasoby – przekształcanie operacji w zasoby, model dojrzałości Richardsona
- [ ] **Zasoby specjalne** – zasoby-kontrolery, transakcje/scalenia jako zasoby, zasoby jednorazowe (tokeny)
- [ ] **HATEOAS** – hipermedia jako silnik stanu aplikacji
- [ ] Niezawodność HTTP – idempotencja, **POST Once Exactly** ([[Technologie internetowe w przetwarzaniu rozproszonym/POST once exactly]] pusta), warunkowe żądania (`If-Match`, `ETag`), wzorzec POST→PUT
- [ ] Opisać zasoby specjalne z własnego projektu ([[Technologie internetowe w przetwarzaniu rozproszonym/TIWPR - Projekt - Dokumentacja usługi]])

### 15. Asynchroniczna komunikacja HTTP, WebSocket 🟡
- [ ] Techniki przed WebSocket: polling, **long polling (Comet)**, streaming, **Server-Sent Events**
- [ ] **Handshake WebSocket** – `Upgrade`, `Sec-WebSocket-Key/Accept`
- [ ] Budowa ramki WebSocket, maskowanie
- [ ] Uzupełnić `#TODO` w [[Technologie internetowe w przetwarzaniu rozproszonym/WebSocket]] (rozszerzenia, podprotokoły, TLS, API)

### 16. Asynchroniczna implementacja serwerów 🔴
- [ ] Model wątek-na-żądanie vs model zdarzeniowy (pętla zdarzeń)
- [ ] Asynchroniczne API serwerowe (np. Servlet 3.0 async, Node.js, Netty/Jetty), callback/future/promise, async/await
- [ ] Problem C10K, porównanie wydajności

---

## Zarządzanie systemami komputerowymi

### 17. Zarządzanie oprogramowaniem 🟢
- [ ] Procedury instalacji / usuwania / aktualizacji pakietu – `#TODO` w [[Zarządzanie Systemami Komputerowymi/Pakiety Oprogramowania]]
- [ ] Plik `.spec` ([[Zarządzanie Systemami Komputerowymi/Plik spec]] pusty), skrypty i wyzwalacze ([[Zarządzanie Systemami Komputerowymi/Wyzwalacze (triggers)]])

### 18. LDAP 🟡
- [ ] **Replikacja danych** – master-slave, multi-master, syncrepl (refreshOnly/refreshAndPersist), slurpd – [[Zarządzanie Systemami Komputerowymi/Syncrepl]] pusta
- [ ] **Active Directory** – domeny, drzewa, lasy, OU, kontrolery domen, GC, replikacja multi-master, Kerberos, GPO (w [[Zaliczenie BSR]] jest tylko skrót GPO)
- [ ] OpenLDAP – `#TODO` w [[Zarządzanie Systemami Komputerowymi/OpenLDAP]]

### 19. Archiwizacja i odtwarzanie 🟡
- [ ] **Schematy archiwizacji**: pełna, przyrostowa, różnicowa; poziomy dump 0–9; rotacje nośników (GFS – dziadek/ojciec/syn, wieża Hanoi)
- [ ] Reguła 3-2-1, RPO/RTO ([[Zarządzanie Systemami Rozproszonymi/IaC]] – RPO `#TODO`)
- [ ] Deduplikacja: plikowa vs blokowa, stały vs zmienny rozmiar bloku, inline vs post-process – zebrać w jednym miejscu
- [ ] Zsync – algorytm (`#TODO` w [[Zarządzanie Systemami Komputerowymi/Zsync]])
- [ ] RAID – obrazki i RAID 1 (`#TODO` w [[Konstrukcja Systemów Chmurowych/RAID]])
- [ ] [[Archiwizacja i odtwarzanie]] – rozwiązania komercyjne `#TODO`

### 20. Rozproszone systemy plików 🔴
- [ ] **Modele dostępu**: upload/download vs dostęp zdalny
- [ ] **Transparentność**: położenia, dostępu, niezależność od położenia, nazewnictwo (montowanie)
- [ ] **Semantyki współbieżnego dostępu**: semantyka UNIX, semantyka sesji, pliki niemodyfikowalne, transakcje
- [ ] Serwery stanowe vs bezstanowe – porównanie (jest tylko [[Zarządzanie Systemami Komputerowymi/Bezstanowy Serwer Plików]])
- [ ] **Replikacja** w rozproszonych systemach plików
- [ ] **NFS** – wersje v2/v3/v4 (v4 stanowy), RPC/XDR, blokady, montowanie
- [ ] **CIFS/SMB** – protokół, oplocks
- [ ] **AFS** – cache całych plików, callbacki, komórki, semantyka sesji
- [ ] **Coda** – praca rozłączna (hoarding, emulation, reintegration), VSG/AVSG, rozwiązywanie konfliktów
- [ ] **GFS** (Global File System – Red Hat, klastrowy) / ewentualnie Google FS – ustalić, który był na wykładzie
- [ ] **OCFS** / OCFS2 – klastrowy system plików na współdzielonym dysku

### 21. Lokalne systemy plików 🟢
- [ ] [[Zarządzanie Systemami Komputerowymi/CoW]] – pusta notatka (treść jest rozproszona po Btrfs/LVM/wykładzie); warto zebrać definicję
- [ ] ZFS – link `[[ZFS]]` w [[Zarządzanie Systemami Komputerowymi/Kopia Migawkowa]] prowadzi donikąd

---

## Systemy wysokiej niezawodności

### 22. Wsteczne odtwarzanie stanu 🟢
- [ ] Definicje: spójny stan globalny, linia odtwarzania, wiadomości osierocone (*orphan*) i zagubione (*lost/in-transit*), **efekt domina** (jest jedno zdanie)
- [ ] Systematyka: punkty kontrolne (skoordynowane / nieskoordynowane / wymuszone komunikacją) vs logowanie komunikatów (pesymistyczne / optymistyczne / przyczynowe) – tabela porównawcza
- [ ] [[Systemy Wysokiej Niezawodności/Checkpoint]] – `#TODO`; [[Systemy Wysokiej Niezawodności/Algorytm Manivannana-Singhala]] – pełen algorytm `#TODO`
- [ ] Koo-Toueg – wyjaśnić „przetwarzanie dyfuzyjne” (`#TODO` w notatce)
- [ ] Output commit – definicja

### 23. Rozproszone uzgadnianie w środowisku zawodnym 🟡
- [ ] **Definicja problemu konsensusu**: zgodność, ważność (validity), terminacja
- [ ] **Twierdzenie FLP** (niemożliwość konsensusu w systemie asynchronicznym z 1 awarią)
- [ ] **Problem bizantyjskich generałów** – Lamport-Shostak-Pease, algorytm OM(m), warunek N > 3f
- [ ] Modele awarii (crash, omission, fail-stop, fail-recovery, bizantyjskie) – zebrać; [[Systemy Wysokiej Niezawodności/Błąd Bizantyjski]] to jedno zdanie
- [ ] Detektory awarii (◇S, P), konsensus z detektorem (Chandra-Toueg)
- [ ] Problem dwóch armii (uzgadnianie przy zawodnych kanałach)
- [ ] Reliable broadcast – link `[[Reliable Broadcast]]` w [[Systemy Wysokiej Niezawodności/Replikacja Procesu]] prowadzi donikąd

### 24. Niezawodne zatwierdzanie transakcji 🟡
- [ ] **2PC** – przeniesić ze [[Slajdy-FT-06_Commit.pdf#page=7|slajdów]] do notatki: fazy, stany koordynatora/uczestników, obsługa timeoutów, blokowanie przy awarii koordynatora
- [ ] **3PC** – stan *pre-commit*, dlaczego nieblokujący, założenia (brak partycji) – obecnie tylko [[Slajdy-FT-06_Commit.pdf#page=18|slajdy]]
- [ ] Protokoły terminacji i odtwarzania (co robi uczestnik po restarcie)
- [ ] [[ACID]] – same nagłówki
- [ ] [[Systemy Wysokiej Niezawodności/2 Phase Locking]] – tylko skrót; opisać fazy wzrostu/zmniejszania, strict 2PL
- [ ] Głosowanie dynamiczne ([[Dynamiczne głosowanie]]), Jajodia-Mutchler ([[Systemy Wysokiej Niezawodności/Algorytm Jajodia-Mutchlera]] pusty)

---

## Bezpieczeństwo systemów rozproszonych

### 25. Mechanizmy polityki uwierzytelniania 🟡
- [ ] **Kerberos** – AS/TGS, TGT, bilety usług, uwierzytelnienie wzajemne ([[Bezpieczeństwo Systemów Rozproszonych/Kerberos]] – 2 linijki)
- [ ] Czynniki uwierzytelniania (wiedza/posiadanie/cecha), MFA/2FA, OTP (HOTP/TOTP)
- [ ] Uwierzytelnianie certyfikatami X.509 / PKI, TLS z uwierzytelnieniem klienta ([[Bezpieczeństwo Systemów Rozproszonych/SSL & TLS]] – stub)
- [ ] 802.1X / EAP (związek z RADIUS)
- [ ] OAuth 2.0 / OpenID Connect
- [ ] Polityki haseł (w PAM tylko nazwy modułów)
- [ ] HTTP Digest – podatności (`#TODO` w [[Bezpieczeństwo Systemów Rozproszonych/HTTP Authentication]])

### 26. SSO 🔴
- [ ] Zasada działania – dostawca tożsamości (IdP) vs dostawca usługi (SP), zaufanie, asercje/bilety
- [ ] Realizacje: Kerberos, SAML 2.0 (przepływ), OpenID Connect, CAS
- [ ] Zastosowania (środowiska korporacyjne, AD, aplikacje webowe)
- [ ] **Zalety** (wygoda, mniej haseł, centralna polityka) i **wady** (SPOF, skutki przejęcia konta, złożoność)

### 27. Mechanizmy polityki kontroli dostępu 🟡
- [ ] Modele: **DAC, MAC, RBAC, ABAC** – definicje i porównanie (w [[Bezpieczeństwo Systemów Rozproszonych/AppArmor]] są tylko w uzasadnieniach odpowiedzi testowych)
- [ ] Macierz dostępu, ACL vs capabilities
- [ ] Modele formalne: Bell-LaPadula, Biba
- [ ] **SELinux** (etykiety, typy) vs AppArmor (ścieżki)
- [ ] Linux capabilities, sudo, sandboxing (seccomp)

### 28. Same Origin i Same Site 🔴
- [ ] Definicja *origin* (schemat + host + port), co SOP ogranicza (DOM, XHR/fetch, cookies)
- [ ] Wyjątki i obejścia: CORS (preflight, nagłówki `Access-Control-*`), `postMessage`, JSONP
- [ ] Definicja *site* (eTLD+1) vs origin
- [ ] Atrybut cookie **SameSite** – `Strict`, `Lax`, `None`; związek z CSRF

### 29. XSS, SQLi, CSRF 🔴
- [ ] **XSS** – reflected, stored, DOM-based; ochrona: kodowanie wyjścia, CSP, `HttpOnly`
- [ ] **SQL Injection** – mechanizm, rodzaje (union, blind, error-based); ochrona: zapytania parametryzowane, ORM, najmniejsze uprawnienia
- [ ] **CSRF** – mechanizm; ochrona: tokeny anty-CSRF, SameSite, sprawdzanie `Origin`/`Referer`
- [ ] OWASP Top 10 – [[OWASP ASVS]] ma same `#TODO`

---

## Zarządzanie systemami rozproszonymi

### 30. Obszary zarządzania siecią 🔴
- [ ] Model **FCAPS** (ISO): Fault, Configuration, Accounting, Performance, Security – definicja i przykłady każdego obszaru
- [ ] Architektura zarządzania: menedżer, agent, baza MIB, protokół

### 31. SNMP 🔴
- [ ] [[SNMP]] – **plik jest pusty**
- [ ] Architektura menedżer–agent, porty 161/162
- [ ] Operacje/PDU: Get, GetNext, GetBulk, Set, Response, Trap, Inform
- [ ] Wersje: v1, v2c (community), v3 (USM, VACM – uwierzytelnianie i szyfrowanie)
- [ ] SMI, kodowanie BER/ASN.1

### 32. MIB 🔴
- [ ] Definicja MIB, drzewo OID (iso.org.dod.internet.mgmt.mib-2, gałąź private/enterprises)
- [ ] SMI – definicja obiektów (`OBJECT-TYPE`, SYNTAX, ACCESS, STATUS), tabele (wiersze, indeksy)
- [ ] MIB-II – grupy (system, interfaces, ip, tcp, udp…)
- [ ] RMON

---

## Konstrukcja systemów chmurowych

### 33. Wirtualizacja 🟢
- [ ] **Migracja maszyn wirtualnych** – cold vs live, *pre-copy* / *post-copy*, migracja pamięci i dysków (`#TODO` w [[Zarządzanie Systemami Rozproszonymi/Wirtualizacja#Punkty kontrolne i migracja]])
- [ ] Nadzorcy typu 1 vs typu 2 – jawne porównanie
- [ ] Binarna translacja (VMware) – szczegóły przy pełnej wirtualizacji
- [ ] Wirtualizacja GPU – passthrough / mediated devices (`#TODO`)
- [ ] [[Konstrukcja Systemów Chmurowych/Open Virtualization Format]] – kilka `#TODO`

### 34. Konteneryzacja 🟢
- [ ] [[Konstrukcja Systemów Chmurowych/OpenVZ]] – zarządzanie zasobami `#TODO`
- [ ] Zalety kopii migawkowych dla kontenerów (`#TODO` w [[Konstrukcja Systemów Chmurowych/Konteneryzacja]])
- [ ] Mechanizmy dodatkowej izolacji: seccomp, capabilities, user namespaces (rootless)

### 35. Przetwarzanie w chmurze 🟡
- [ ] **Chmura a grid** – porównanie (`#TODO` w [[IaaS#Grids vs IaaS]])
- [ ] **Chmury hybrydowe** (i publiczne, community) – brak; [[Cloud Types]] dotyczy typów danych, nie modeli wdrożenia
- [ ] **(Auto)skalowanie** – reguły, metryki, skalowanie reaktywne vs predykcyjne, HPA ([[Scaling on-demand]] – `#TODO`)
- [ ] **Standaryzacja** – CDMI, OCCI, OVF, TOSCA; problem vendor lock-in
- [ ] **Bezpieczeństwo** – model współodpowiedzialności, multi-tenancy, izolacja (sekcje w [[Konstrukcja Systemów Chmurowych/Cloud Computing]] z `#TODO`)
- [ ] [[Konstrukcja Systemów Chmurowych/Virtual Private Cloud]] – pusta

### 36. Systemy składowania danych 🟡
- [ ] **iSCSI** – initiator, target, LUN, IQN, porównanie z FC
- [ ] **Multipath I/O** – nadmiarowe ścieżki, failover/load balancing, `dm-multipath`
- [ ] **OCFS2** – klastrowy system plików na współdzielonym urządzeniu blokowym, DLM
- [ ] **DRBD** – replikacja blokowa przez sieć (RAID-1 sieciowy), protokoły A/B/C, primary/secondary
- [ ] Ceph – `#TODO` (obrazki PG, journal, monitor, block storage [[Konstrukcja Systemów Chmurowych/Ceph block storage]] pusty)

---

## Metody bezpiecznego programowania

### 37. Monitory w C# / Java 🟡
- [ ] **Java**: `synchronized` (metody/bloki), `wait()`/`notify()`/`notifyAll()`, jedna niejawna zmienna warunkowa, *spurious wakeups* (pętla `while`)
- [ ] `java.util.concurrent.locks`: `ReentrantLock` + `Condition`
- [ ] **C#**: `lock`, `Monitor.Enter/Exit/Wait/Pulse/PulseAll`
- [ ] Semantyka sygnalizacji: Hoare (signal-and-urgent-wait) vs Mesa (signal-and-continue) – czym różni się od [[Klasyczny Monitor|monitora klasycznego]]
- [ ] Przykład kodu (np. bufor ograniczony) – `#TODO` w notatce

### 38. Algorytm Eraser 🔴
- [ ] Definicja wyścigu danych (*data race*)
- [ ] Algorytm **lockset**: C(v) := C(v) ∩ locks_held(t), zgłoszenie przy C(v) = ∅
- [ ] Maszyna stanów zmiennej: Virgin → Exclusive → Shared → Shared-Modified (inicjalizacja, dane tylko do odczytu)
- [ ] Obsługa blokad odczyt/zapis
- [ ] Ograniczenia: fałszywe alarmy, dynamiczna analiza (tylko wykonane ścieżki)

### 39. High-level data race 🔴
- [ ] Definicja wyścigu wysokiego poziomu (niespójny dostęp do grupy powiązanych zmiennych mimo braku klasycznego wyścigu)
- [ ] Algorytm Artho-Havelunda-Biddle'a: **widoki** (*views*) i **zgodność widoków** (*view consistency*)
- [ ] Przykład (np. współrzędne x,y chronione w osobnych blokach)
- [ ] Porównanie z Eraserem

### 40. Model checking – Java Pathfinder 🔴
- [ ] Idea sprawdzania modelu: przestrzeń stanów, własności (bezpieczeństwo, żywotność, LTL)
- [ ] Problem eksplozji stanów; techniki redukcji (redukcja częściowego porządku, abstrakcja, haszowanie stanów)
- [ ] **JPF**: własna JVM, eksploracja przeplotów wątków i nieterminizmu, backtracking, wykrywane błędy (zakleszczenia, nieobsłużone wyjątki, asercje, wyścigi)
- [ ] Listenery, rozszerzenia (symbolic PathFinder)

### 41. Pamięć transakcyjna 🔴
- [ ] Idea: atomowe bloki `atomic {}`, optymistyczna współbieżność
- [ ] Implementacja: **STM** (programowa) vs **HTM** (Intel TSX), wersjonowanie danych (eager/lazy), wykrywanie konfliktów (eager/lazy), log undo/redo
- [ ] Przykłady: TL2, Clojure refs, Haskell STM
- [ ] Zastosowanie
- [ ] **Zalety** (kompozycyjność, brak zakleszczeń) i **wady** (operacje I/O, narzut, livelock, słaba izolacja)

### 42. Linearizability 🟢
- (brak istotnych braków; ewentualnie przykład historii linearyzowalnej i nielinearyzowalnej rozrysowany)

### 43. Model pamięci w Javie 🔴
- [ ] **JMM**: pamięć główna vs kopie robocze wątków, widoczność zmian
- [ ] Relacja **happens-before**
- [ ] `volatile` (widoczność, ale nie atomowość), `synchronized`, `final`
- [ ] Double-checked locking – problem i poprawka (JSR-133)
- [ ] Relaxed memory models / TSO (x86) – rozwinąć wzmiankę z [[Synchronizacja]]

### 44. Cilk 🔴
- [ ] Słowa kluczowe: `spawn`, `sync` (`cilk_spawn`, `cilk_sync`, `cilk_for`)
- [ ] Model obliczeń jako **DAG**: praca W₁, rozpiętość W∞ (*span*), przyspieszenie, równoległość
- [ ] **Work stealing** – deque na wątek, kradzież z dołu kolejki, gwarancje czasowe/pamięciowe
- [ ] Przykład: Fibonacci

### 45. Model aktorów i rachunek pi 🔴
- [ ] **Model aktorów** (Hewitt): aktor, skrzynka pocztowa, asynchroniczne komunikaty, zachowania (wyślij/utwórz/zmień zachowanie), brak współdzielonego stanu; Erlang, Akka
- [ ] **Rachunek pi** (Milner): składnia (prefiksy wysyłania/odbioru, kompozycja równoległa, restrykcja ν, replikacja), reguła redukcji, mobilność kanałów
- [ ] Porównanie z modelem aktorów

### 46. CRDT i typy chmurowe 🟡
- [ ] Definicja **ostatecznej spójności** (eventual / strong eventual consistency)
- [ ] **CRDT**: state-based (CvRDT – półkrata, join) vs operation-based (CmRDT – operacje komutatywne)
- [ ] Przykłady: G-Counter, PN-Counter, G-Set, 2P-Set, OR-Set, LWW-Register
- [ ] **Typy chmurowe** (Burckhardt et al.) – rozwinąć [[Cloud Types]]: typy (CInt, CString, CSet), `yield`, `flush`, model rewizji (*revision diagrams*)

---

## Systemy rozproszone dużej skali

### 47. Architektury systemów dużej skali 🟡
- [ ] **Klasyfikacje**: klastry, gridy, chmury, P2P, systemy mobilne/IoT, fog/edge – idea działania i zastosowania każdej klasy
- [ ] Przykłady implementacji (np. BOINC, Globus, Dynamo, Cassandra)
- [ ] Rodzaje architektur w [[Systemy Rozproszone Dużej Skali/Big Data#Rodzaje architektur]] – rozwinąć punkty (teraz 1 linijka)

### 48. Nieustrukturyzowane P2P 🔴
- [ ] Architektury: scentralizowana (Napster), zdecentralizowana (Gnutella), hybrydowa / superwęzły (KaZaA/FastTrack)
- [ ] Strategie wyszukiwania: flooding (TTL), expanding ring, random walk (k-walkers), wyszukiwanie z indeksami / replikacja zasobów
- [ ] Wady (skalowalność, brak gwarancji znalezienia)

### 49. Ustrukturyzowane P2P, DHT 🔴
- [ ] Definicja **DHT** – interfejs put/get, spójne haszowanie
- [ ] **Chord** – pierścień, finger table, routing O(log N), dołączanie/odchodzenie
- [ ] **Pastry** (routing prefiksowy), **Kademlia** (metryka XOR, k-kubełki), **CAN** (przestrzeń d-wymiarowa)
- [ ] Przykłady systemów (BitTorrent DHT, Dynamo, IPFS)

### 50. BitTorrent 🔴
- [ ] Przeznaczenie – efektywna dystrybucja dużych plików
- [ ] Elementy: plik `.torrent`, tracker (i trackerless DHT), seed/leecher, swarm, fragmenty (*pieces*) i hasze
- [ ] Algorytmy: **rarest first**, **tit-for-tat / choking**, optimistic unchoking, endgame mode

### 51. Big Data, NoSQL, CAP, PACELC 🟡
- [ ] **Źródła dużych danych** (IoT, logi, media społecznościowe, transakcje) – brak jawnej listy
- [ ] **Teoria CAP** – treść twierdzenia, dowód szkicowo, przykłady systemów CP/AP ([[Systemy Rozproszone Dużej Skali/Teoria CAP]] – stub)
- [ ] **PACELC** – przy partycji A vs C, w przeciwnym razie latency vs consistency; klasyfikacja systemów (PA/EL Dynamo, PC/EC HBase…)
- [ ] NoSQL – uzupełnić `#TODO`: document, graph, key-value (w [[Systemy Rozproszone Dużej Skali/noSQL]]); BASE vs ACID
- [ ] [[Systemy Rozproszone Dużej Skali/Dynamo]] (pusta), [[Systemy Rozproszone Dużej Skali/BigTable]] (model danych `#TODO`)
- [ ] [[Sharding]] – `#TODO`

### 52. Architektura Big Data 🟢
- [ ] Konkretne narzędzia na etapach: pozyskiwanie (Kafka, Flume, NiFi), składowanie (HDFS, S3, data lake/warehouse), przetwarzanie (MapReduce, Spark, Flink)

### 53. Hadoop i Spark 🔴
- [ ] **Hadoop**: HDFS (NameNode, DataNode, bloki, replikacja), YARN (ResourceManager, NodeManager), MapReduce (map, shuffle/sort, reduce, combiner)
- [ ] **Spark**: RDD (niemutowalność, lineage), transformacje leniwe vs akcje, DAG scheduler, DataFrame/Spark SQL, Streaming
- [ ] Porównanie Spark vs MapReduce (pamięć, iteracje)

### 54. Blockchain 🔴
- [ ] Budowa: bloki, łańcuch haszy, drzewo Merkle, transakcje, podpisy cyfrowe
- [ ] Konsensus: **Proof of Work**, Proof of Stake, (P)BFT w sieciach permissioned
- [ ] Rodzaje: publiczny / prywatny / konsorcjalny
- [ ] Przeznaczenie: kryptowaluty, smart kontrakty (Ethereum), łańcuchy dostaw

---

## Programowanie sieciowe

### 55. Obsługa interfejsów sieciowych 🟢
- [ ] Odpowiedzi na pytania z [[Programowanie Sieciowe/Obsługa Interfejsów Sieciowych#Pytania i zadania]] (puste)
- [ ] Pełna lista żądań `ioctl` dla interfejsów (`SIOCGIFFLAGS`/`SIOCSIFFLAGS`, `SIOCGIFADDR`/`SIOCSIFADDR`, `SIOCGIFMTU`…), struktura `ifreq`, `getifaddrs(3)`

### 56. Tablica routingu i ARP 🟢
- [ ] Tablice `local`/`main` – `#TODO prez 4-5/32` w [[Programowanie Sieciowe/Tablica Routingu]]
- [ ] Konfiguracja dodatkowej bramy – `#TODO` w [[Programowanie Sieciowe/ip(8)]]

### 57. PF_PACKET 🟢
- [ ] Odpowiedzi na pytania 2 i 3 w [[Programowanie Sieciowe/Gniazda Sieciowe PF_PACKET#Pytania i zadania]]
- [ ] Opisać przykład odbioru ramki (`#TODO`)

### 58. PF_NETLINK 🟡
- [ ] Uzupełnić `#TODO prez 17–27/32`: struktura `rtmsg`, `rtattr` i makra `RTA_*`/`NLMSG_*`, pełny przykład pobrania tablicy routingu
- [ ] Rodziny netlink (NETLINK_ROUTE, NETLINK_KOBJECT_UEVENT…), grupy multicast
- [ ] Odpowiedzi na pytania (s. 31/32)

### 59. Nieprzetworzone gniazda 🟡
- [ ] [[Nieprzetworzone Gniazda Sieciowe]] – sekcje „Pakiety transmitowane” i przykład transmisji IPv4 puste
- [ ] Opcja `IP_HDRINCL`, uprawnienia (`CAP_NET_RAW`), przykład ICMP (ping)
- [ ] [[Programowanie Sieciowe/Struktura nagłówka pakietów IPv4]] – `#TODO wszystko z prez`

### 60. Obsługa operacji I/O komunikacji sieciowej 🔴
- [ ] Modele I/O (Stevens): **blokujące, nieblokujące (`O_NONBLOCK`), multipleksacja (`select`, `poll`, `epoll`), sterowane sygnałami (`SIGIO`/`FIOASYNC`), asynchroniczne (`aio_*`, `io_uring`)**
- [ ] `epoll` – tryby level-triggered vs edge-triggered
- [ ] Porównanie modeli (tabela)

### 61. Architektury serwerów sieciowych 🔴
- [ ] Serwer **iteracyjny** vs **współbieżny**
- [ ] Proces na klienta (`fork`), **prefork** (pula procesów), wątek na klienta, **pula wątków** (prethreading)
- [ ] Serwer **zdarzeniowy** z multipleksacją (wzorzec Reactor / Proactor)
- [ ] Architektury hybrydowe (np. nginx: master + workery z epoll), porównanie wydajności i skalowalności
