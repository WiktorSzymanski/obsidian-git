---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 31
---
# 31. Protokół SNMP
---
> **SNMP** (_Simple Network Management Protocol_) to protokół warstwy aplikacji z rodziny TCP/IP. Służy do **monitorowania i zarządzania urządzeniami sieciowymi** (routery, przełączniki, serwery, drukarki, UPS) w modelu **zarządca–agent**. Zarządca odczytuje i modyfikuje wartości **obiektów zarządzanych** zdefiniowanych w bazie **MIB**, a agent może sam wysyłać **powiadomienia** o zdarzeniach.

Obszary zarządzania: [[30 Obszary zarządzania siecią komputerową]]. Baza informacji: [[32 Baza informacji zarządzania MIB]].

## Architektura (Internet-standard Management Framework)
Trzy elementy:
1. **SMI** (_Structure of Management Information_) – język definiowania obiektów (podzbiór ASN.1),
2. **MIB** – bazy definicji obiektów zarządzanych,
3. **protokół SNMP** – wymiana komunikatów.

Uczestnicy:
- **zarządca** (_manager_, NMS) – wysyła żądania, odbiera powiadomienia,
- **agent** – w urządzeniu, odpowiada na żądania, wysyła powiadomienia,
- w SNMPv3: **jednostka SNMP** (_SNMP entity_) = **silnik** (_engine_: dyspozytor, podsystem przetwarzania komunikatów, podsystem bezpieczeństwa, kontrola dostępu) + **aplikacje** (generator poleceń, odpowiadacz, generator/odbiorca powiadomień, proxy).

### Transport
- **UDP** (bezpołączeniowy – mały narzut, działa także przy przeciążonej/uszkodzonej sieci; niezawodność przez timeouty i retransmisje zarządcy),
- **port 161** – agent (żądania), **port 162** – zarządca (odbiór trap/inform),
- możliwe: TCP, TLS/DTLS (RFC 6353, port 10161/10162).

## Wersje
| Wersja | Dokumenty | Bezpieczeństwo | Nowości |
|---|---|---|---|
| **SNMPv1** (1988/1990) | RFC 1155 (SMIv1), 1157, 1213 (MIB-II) | **community string** jawnym tekstem | Get, GetNext, Set, Trap |
| **SNMPv2c** (1996) | RFC 1901–1908, SMIv2 RFC 2578 | community (jak v1) | **GetBulk**, **Inform**, SNMPv2-Trap, liczniki **Counter64**, lepsze kody błędów i wyjątki |
| **SNMPv3** (1998/2002) | RFC 3411–3418 | **USM** (uwierzytelnianie, szyfrowanie), **VACM** (kontrola dostępu) | modularna architektura, engineID |

(SNMPv2p/v2u – historyczne warianty z nieprzyjętym modelem bezpieczeństwa; „c” = _community-based_.)

## Operacje (jednostki PDU)
| PDU | Kierunek | Opis |
|---|---|---|
| **GetRequest** | zarządca → agent | odczyt wartości wskazanych instancji obiektów (lista OID) |
| **GetNextRequest** | zarządca → agent | odczyt **następnej** w porządku leksykograficznym instancji po podanym OID – przeglądanie tablic i nieznanych drzew (**walk**) |
| **GetBulkRequest** (v2c+) | zarządca → agent | wiele kolejnych GetNext w jednym żądaniu: parametry **non-repeaters** (ile pierwszych zmiennych jak zwykły GetNext) i **max-repetitions** (ile kolejnych wierszy dla pozostałych) – efektywne pobieranie tablic |
| **SetRequest** | zarządca → agent | zmiana wartości obiektów (konfiguracja, np. `ifAdminStatus = down`); atomowa dla całej listy zmiennych |
| **Response** (v1: **GetResponse**) | agent → zarządca | odpowiedź na Get/GetNext/GetBulk/Set/Inform |
| **Trap** (v1) / **SNMPv2-Trap** | agent → zarządca | **niepotwierdzane** asynchroniczne powiadomienie o zdarzeniu |
| **InformRequest** (v2c+) | agent → zarządca (lub zarządca → zarządca) | powiadomienie **potwierdzane** odpowiedzią – retransmitowane do skutku |
| **Report** (v3) | – | komunikaty wewnętrzne silnika (np. odkrywanie engineID, błędy synchronizacji czasu) |

### Format komunikatu SNMPv1/v2c
```
Komunikat SNMP (ASN.1 SEQUENCE)
├─ version     INTEGER  (0 = v1, 1 = v2c, 3 = v3)
├─ community   OCTET STRING  ("public")
└─ PDU
   ├─ request-id    INTEGER   (dopasowanie odpowiedzi do żądania)
   ├─ error-status  INTEGER
   ├─ error-index   INTEGER   (która zmienna spowodowała błąd)
   └─ variable-bindings  SEQUENCE OF { name OID, value }
```
W GetBulk pola error-status/error-index zastąpione przez non-repeaters/max-repetitions.

**Trap v1** ma inny format: `enterprise` (OID), `agent-addr`, **`generic-trap`**:
- 0 coldStart, 1 warmStart, 2 **linkDown**, 3 **linkUp**, 4 authenticationFailure, 5 egpNeighborLoss, 6 enterpriseSpecific,
- `specific-trap`, `time-stamp` (sysUpTime), varbinds.

**SNMPv2-Trap** – zwykły format PDU, dwie pierwsze zmienne: `sysUpTime.0` i `snmpTrapOID.0` (identyfikator powiadomienia, np. `linkDown` = 1.3.6.1.6.3.1.1.5.3).

### Kodowanie
**BER** (_Basic Encoding Rules_ ASN.1) – każdy element jako **TLV** (_Type-Length-Value_). Np. INTEGER 5 → `02 01 05`, OCTET STRING „public” → `04 06 70 75 62 6C 69 63`, SEQUENCE → `30 …`.

### Kody błędów
- v1: `noError(0)`, `tooBig(1)` (odpowiedź nie mieści się w komunikacie), `noSuchName(2)`, `badValue(3)`, `readOnly(4)`, `genErr(5)`,
- v2c dodaje: `noAccess`, `wrongType`, `wrongLength`, `wrongEncoding`, `wrongValue`, `noCreation`, `inconsistentValue`, `resourceUnavailable`, `commitFailed`, `undoFailed`, `authorizationError`, `notWritable`, `inconsistentName`,
- v2c **wyjątki** w varbindach (zamiast błędu całego żądania): `noSuchObject`, `noSuchInstance`, `endOfMibView` (koniec drzewa przy GetNext/GetBulk).

## Przykładowa sesja
```bash
# odczyt opisu systemu i czasu działania
snmpget -v2c -c public 192.168.1.1 SNMPv2-MIB::sysDescr.0 SNMPv2-MIB::sysUpTime.0
# SNMPv2-MIB::sysDescr.0 = STRING: Cisco IOS Software, ...
# DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (123456) 0:20:34.56

# przejście drzewa interfejsów (GetNext / GetBulk)
snmpwalk -v2c -c public 192.168.1.1 IF-MIB::ifDescr
snmpbulkwalk -v2c -c public 192.168.1.1 IF-MIB::ifTable

# zmiana wartości (wyłączenie interfejsu 3)
snmpset -v2c -c private 192.168.1.1 IF-MIB::ifAdminStatus.3 i 2

# odbiór powiadomień
snmptrapd -f -Lo

# SNMPv3 z uwierzytelnianiem SHA i szyfrowaniem AES
snmpget -v3 -l authPriv -u admin -a SHA -A 'hasloAuth123' -x AES -X 'hasloPriv123' 192.168.1.1 sysName.0
```
Działanie `snmpwalk`: `GetNext(ifDescr)` → `ifDescr.1 = "eth0"`, `GetNext(ifDescr.1)` → `ifDescr.2`, …, aż odpowiedź wyjdzie poza poddrzewo `ifDescr`.

## SNMPv1/v2c – model bezpieczeństwa oparty na community
- **Community string** – współdzielone „hasło” przesyłane **jawnym tekstem** w każdym komunikacie; zwykle `public` (odczyt) i `private` (zapis) – domyślne wartości są częstą luką.
- Agent może ograniczać dostęp listą adresów zarządców (łatwe do sfałszowania w UDP).
- Brak integralności, poufności, ochrony przed powtórzeniem.

## SNMPv3 – bezpieczeństwo
### USM – _User-based Security Model_ (RFC 3414)
- **Użytkownicy** z kluczami wyprowadzonymi z haseł i **lokalizowanymi** do `snmpEngineID` agenta (kradzież klucza z jednego urządzenia nie daje dostępu do innych).
- **Uwierzytelnianie i integralność**: HMAC-MD5-96, HMAC-SHA-96 (nowsze RFC 7860: HMAC-SHA-2).
- **Poufność**: CBC-DES (przestarzałe), CFB-AES-128 (RFC 3826).
- **Ochrona przed powtórzeniem / opóźnieniem**: `snmpEngineBoots` i `snmpEngineTime` – komunikat odrzucany, jeśli różnica czasu > **150 s**; mechanizm odkrywania (discovery) engineID i czasu przez Report.
- **Poziomy bezpieczeństwa**: `noAuthNoPriv`, `authNoPriv`, `authPriv`.

### VACM – _View-based Access Control Model_ (RFC 3415)
- **Grupy** (użytkownik + model bezpieczeństwa → grupa),
- **widoki MIB** (_views_) – poddrzewa OID z maskami, włączane/wyłączane,
- reguły dostępu: grupa + kontekst + poziom bezpieczeństwa → widok do **odczytu**, **zapisu**, **powiadomień**.

## Zalety i wady SNMP
✔ prosty, lekki, powszechnie wspierany przez praktycznie wszystkie urządzenia sieciowe; standardowe MIB-y; odporny (UDP) w sytuacji awarii sieci.
✘ v1/v2c niebezpieczne; `Set` rzadko używany do konfiguracji (brak transakcji na wielu urządzeniach, słaba ekspresja) → NETCONF/YANG; odpytywanie nie skaluje się przy dużej częstotliwości i dużej liczbie metryk → telemetria strumieniowa (gNMI); ograniczone typy danych; podatność na **ataki wzmacniające DDoS** (GetBulk z fałszywym adresem źródłowym – mały pakiet, duża odpowiedź) – nie wystawiać SNMP do Internetu.
