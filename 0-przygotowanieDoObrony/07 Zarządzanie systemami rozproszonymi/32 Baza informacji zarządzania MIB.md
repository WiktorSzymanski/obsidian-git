---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 32
---
# 32. Baza informacji zarządzania MIB
---
> **MIB** (_Management Information Base_) to **wirtualna baza** (zbiór definicji) **obiektów zarządzanych**, które agent SNMP udostępnia o urządzeniu: parametry konfiguracyjne, liczniki, stany, tablice. MIB nie jest fizyczną bazą danych – to **schemat**. Opisuje **nazwę** (OID), **typ**, **dostęp** i **znaczenie** każdego obiektu, a wartości pochodzą z bieżącego stanu urządzenia.

Protokół dostępu: [[31 Protokół SNMP]].

## SMI – struktura informacji zarządzania
**SMI** definiuje reguły opisu obiektów MIB – jest to **podzbiór ASN.1** (_Abstract Syntax Notation One_) z makrami.
- **SMIv1** – RFC 1155, 1212 (makro `OBJECT-TYPE`),
- **SMIv2** – RFC 2578 (definicje), 2579 (**konwencje tekstowe**), 2580 (zgodność – `OBJECT-GROUP`, `MODULE-COMPLIANCE`).

### Typy danych SMIv2
| Typ | Opis |
|---|---|
| `INTEGER` / `Integer32` | liczba całkowita 32-bit (także wyliczenia `{ up(1), down(2) }`) |
| `Unsigned32` | liczba nieujemna |
| `OCTET STRING` | ciąg bajtów (tekst, adresy MAC) |
| `OBJECT IDENTIFIER` | identyfikator obiektu (OID) |
| `IpAddress` | adres IPv4 (4 bajty) |
| `Counter32`, `Counter64` | **licznik** monotonicznie rosnący, zawija się po maksimum (znaczenie ma różnica między odczytami – np. `ifInOctets`) |
| `Gauge32` | **wskaźnik** – wartość rosnąca i malejąca (np. `ifSpeed`, długość kolejki) |
| `TimeTicks` | czas w setnych sekundy (np. `sysUpTime`) |
| `Opaque` | dowolne dane ASN.1 (przestarzałe) |
| `BITS` | zbiór flag |

**Konwencje tekstowe** (_TEXTUAL-CONVENTION_) – typy z semantyką: `DisplayString` (tekst ASCII ≤ 255), `PhysAddress`, `MacAddress`, `TruthValue`, `RowStatus`, `TimeStamp`, `DateAndTime`, `InetAddress`.

## Drzewo identyfikatorów obiektów (OID)
Obiekty są liśćmi **globalnego, hierarchicznego drzewa rejestracji** (ISO/ITU-T). Każdy węzeł ma numer i etykietę; OID to ścieżka od korzenia, np. `1.3.6.1.2.1.1.1`. Mechanizm OID wspólny z LDAP – [[Zarządzanie Systemami Komputerowymi/OID]].

```
root
├── ccitt(0)
├── iso(1)
│   └── org(3)
│       └── dod(6)
│           └── internet(1)                         1.3.6.1
│               ├── directory(1)                    1.3.6.1.1
│               ├── mgmt(2)                         1.3.6.1.2
│               │   └── mib-2(1)                    1.3.6.1.2.1
│               │       ├── system(1)
│               │       ├── interfaces(2)
│               │       ├── at(3)
│               │       ├── ip(4)
│               │       ├── icmp(5)
│               │       ├── tcp(6)
│               │       ├── udp(7)
│               │       ├── egp(8)
│               │       ├── transmission(10)
│               │       ├── snmp(11)
│               │       ├── host(25)  (HOST-RESOURCES-MIB)
│               │       ├── ifMIB(31) (IF-MIB, ifXTable – liczniki 64-bit)
│               │       └── rmon(16)
│               ├── experimental(3)                 1.3.6.1.3
│               ├── private(4)                      1.3.6.1.4
│               │   └── enterprises(1)              1.3.6.1.4.1
│               │       ├── cisco(9)
│               │       ├── hp(11)
│               │       └── ... (numery PEN nadawane przez IANA)
│               ├── security(5)
│               └── snmpV2(6)                       1.3.6.1.6 (moduły SNMPv2/v3, powiadomienia)
└── joint-iso-ccitt(2)
```
- **mgmt/mib-2** – standardowe obiekty zdefiniowane przez IETF,
- **private.enterprises** – obiekty specyficzne dla producenta; firma uzyskuje **PEN** (_Private Enterprise Number_) od IANA i zarządza swoim poddrzewem,
- **experimental** – obiekty w trakcie standaryzacji.

### Instancje obiektów
- **Obiekt skalarny** (jedna wartość) – instancja z sufiksem **`.0`**: `sysDescr.0` = `1.3.6.1.2.1.1.1.0`.
- **Kolumna tablicy** – instancje identyfikowane **indeksem** wiersza: `ifDescr.2` = `1.3.6.1.2.1.2.2.1.2.2` (drugi interfejs), indeks może być złożony, np. adres IP w `ipNetToMediaTable`.
- Porządek **leksykograficzny** OID pozwala przechodzić drzewo operacją GetNext.

## MIB-II (RFC 1213) – najważniejsze grupy
### system (1.3.6.1.2.1.1)
| Obiekt | OID | Typ / dostęp | Znaczenie |
|---|---|---|---|
| `sysDescr` | .1.1 | DisplayString, RO | opis urządzenia (sprzęt, SO, wersja) |
| `sysObjectID` | .1.2 | OID, RO | identyfikator modelu w poddrzewie producenta |
| `sysUpTime` | .1.3 | TimeTicks, RO | czas od restartu agenta |
| `sysContact` | .1.4 | DisplayString, RW | osoba kontaktowa |
| `sysName` | .1.5 | DisplayString, RW | nazwa (FQDN) |
| `sysLocation` | .1.6 | DisplayString, RW | lokalizacja fizyczna |
| `sysServices` | .1.7 | INTEGER, RO | warstwy OSI obsługiwane (bity) |

### interfaces (1.3.6.1.2.1.2)
- `ifNumber` – liczba interfejsów,
- **`ifTable`** → `ifEntry` (indeks `ifIndex`): `ifDescr`, `ifType` (ethernetCsmacd(6)…), `ifMtu`, `ifSpeed` (Gauge32), `ifPhysAddress`, **`ifAdminStatus`** (RW: up/down/testing – stan żądany), **`ifOperStatus`** (RO: stan faktyczny), `ifLastChange`, **`ifInOctets`/`ifOutOctets`** (Counter32), `ifInErrors`, `ifOutDiscards`…
- Liczniki 32-bitowe przepełniają się na szybkich łączach (1 Gb/s w ~34 s) → **`ifXTable`** z IF-MIB: `ifHCInOctets` (Counter64), `ifName`, `ifAlias`, `ifHighSpeed`.

Pozostałe grupy: **at** (tablica ARP, przestarzała), **ip** (`ipForwarding`, liczniki pakietów, `ipAddrTable`, `ipRouteTable`, `ipNetToMediaTable` – ARP), **icmp** (liczniki komunikatów), **tcp** (`tcpConnTable` – połączenia, `tcpActiveOpens`, `tcpRetransSegs`), **udp** (`udpTable` – nasłuchujące porty), **snmp** (statystyki samego agenta).

Wykorzystanie np. do pomiaru przepustowości: $\text{bps} = \dfrac{(ifInOctets_{t_2} - ifInOctets_{t_1}) \cdot 8}{t_2 - t_1}$ (z uwzględnieniem zawinięcia licznika).

## Definicja obiektu – makro OBJECT-TYPE
```asn1
IF-MIB DEFINITIONS ::= BEGIN
IMPORTS MODULE-IDENTITY, OBJECT-TYPE, Counter32, Gauge32, mib-2 FROM SNMPv2-SMI
        DisplayString FROM SNMPv2-TC;

ifTable OBJECT-TYPE
    SYNTAX      SEQUENCE OF IfEntry
    MAX-ACCESS  not-accessible
    STATUS      current
    DESCRIPTION "A list of interface entries."
    ::= { interfaces 2 }

ifEntry OBJECT-TYPE
    SYNTAX      IfEntry
    MAX-ACCESS  not-accessible
    STATUS      current
    DESCRIPTION "An entry containing management information applicable to a particular interface."
    INDEX       { ifIndex }
    ::= { ifTable 1 }

IfEntry ::= SEQUENCE {
    ifIndex        InterfaceIndex,
    ifDescr        DisplayString,
    ifAdminStatus  INTEGER,
    ifInOctets     Counter32
    -- ...
}

ifAdminStatus OBJECT-TYPE
    SYNTAX      INTEGER { up(1), down(2), testing(3) }
    MAX-ACCESS  read-write
    STATUS      current
    DESCRIPTION "The desired state of the interface."
    ::= { ifEntry 7 }
END
```
Klauzule:
- **SYNTAX** – typ danych,
- **MAX-ACCESS** (SMIv2; SMIv1: ACCESS) – `not-accessible` (tablice, wiersze, indeksy), `accessible-for-notify`, `read-only`, `read-write`, `read-create` (kolumny tworzonych wierszy),
- **STATUS** – `current`, `deprecated`, `obsolete` (SMIv1: `mandatory`, `optional`),
- **DESCRIPTION** – semantyka (tekst normatywny),
- **UNITS**, **REFERENCE**, **DEFVAL** (wartość domyślna),
- **INDEX** / **AUGMENTS** (dla wierszy tablic),
- `::= { rodzic numer }` – umiejscowienie w drzewie OID.

Inne makra SMIv2: **MODULE-IDENTITY** (metadane modułu: organizacja, kontakt, rewizje), **OBJECT-IDENTITY**, **NOTIFICATION-TYPE** (definicja powiadomień/trapów, np. `linkDown` z obiektami `ifIndex`, `ifAdminStatus`, `ifOperStatus`), **OBJECT-GROUP**, **MODULE-COMPLIANCE**.

## Tablice koncepcyjne i tworzenie wierszy
- Tablica = obiekt `SEQUENCE OF Entry` → wiersz `Entry` z klauzulą `INDEX` → kolumny. Tablice i wiersze nie są bezpośrednio dostępne, dostępne są **komórki** (kolumna.indeks).
- Dodawanie/usuwanie wierszy (np. reguły, sondy RMON) przez kolumnę **`RowStatus`** (RFC 2579):
  - `createAndGo(4)` – utwórz i aktywuj od razu,
  - `createAndWait(5)` – utwórz nieaktywny, uzupełnij kolumny, potem `active(1)`,
  - `notInService(2)`, `notReady(3)` (stan), `destroy(6)` – usuń.

## Moduły MIB
- pliki tekstowe z definicjami ładowane przez zarządcę (tłumaczenie OID ↔ nazwy, typy): `SNMPv2-MIB`, `IF-MIB` (RFC 2863), `IP-MIB`, `TCP-MIB`, `HOST-RESOURCES-MIB` (CPU, pamięć, dyski, procesy), `ENTITY-MIB` (komponenty fizyczne), `BRIDGE-MIB`, `Q-BRIDGE-MIB` (VLAN), `UPS-MIB`, `IP-FORWARD-MIB`, MIB-y producentów (`CISCO-*-MIB`),
- narzędzia: `snmptranslate -On IF-MIB::ifDescr` (nazwa → OID), `snmptranslate -Tp` (drzewo), kompilatory MIB w NMS.

## RMON
**RMON** (_Remote Network Monitoring_, RFC 2819 – RMON1, RFC 4502 – RMON2) – MIB dla **sond monitorujących** (agent RMON w przełączniku/sondzie), które **samodzielnie** zbierają i analizują statystyki segmentu sieci. Zarządca nie musi ciągle odpytywać; dane dostępne nawet przy utracie łączności z NMS.

**Grupy RMON1** (warstwy 1–2, Ethernet):
1. **statistics** – liczniki segmentu (pakiety, bajty, broadcasty, kolizje, błędy CRC, rozkład rozmiarów ramek),
2. **history** – próbki statystyk w zadanych odstępach czasu,
3. **alarm** – progi (rosnący/opadający) na dowolnej zmiennej MIB, z histerezą,
4. **host** – statystyki per adres MAC,
5. **hostTopN** – ranking hostów wg wybranej statystyki,
6. **matrix** – ruch między parami adresów (kto z kim),
7. **filter** – definicje filtrów pakietów,
8. **capture** – przechwytywanie pakietów spełniających filtr,
9. **event** – akcje na zdarzenie (log, trap),
10. **tokenRing** – rozszerzenia dla Token Ring.

**RMON2** dodaje warstwy **sieciową i aplikacji**: `protocolDir`, `protocolDist` (rozkład ruchu wg protokołów), `addressMap`, `nlHost`/`nlMatrix` (IP), `alHost`/`alMatrix` (aplikacje), `userHistory`, `probeConfig`.

W praktyce często zastępowane przez NetFlow/IPFIX i telemetrię – [[30 Obszary zarządzania siecią komputerową#P – zarządzanie wydajnością]].
