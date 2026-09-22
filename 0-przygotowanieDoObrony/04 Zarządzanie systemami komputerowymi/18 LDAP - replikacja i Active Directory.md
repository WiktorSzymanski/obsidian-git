---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 18
---
# 18. Usługa katalogowa LDAP – replikacja danych i Active Directory
---
> Uzupełnienie [[Zarządzanie Systemami Komputerowymi/LDAP]] (struktura DIT, schematy, wyszukiwanie, operacje, partycjonowanie są tam opisane) o **replikację danych** oraz **Active Directory**.

## Przypomnienie – partycjonowanie a replikacja
- **Partycjonowanie** (_referrals_, _chaining_) – różne poddrzewa DIT na różnych serwerach → skalowalność zapisu i delegacja administracji ([[Zarządzanie Systemami Komputerowymi/Odwołania do innych serwerów#Partycjonowanie przestrzeni nazw]]).
  - **referral** – serwer odsyła klientowi adres innego serwera (klient ponawia zapytanie),
  - **chaining** – serwer sam przekazuje zapytanie dalej i zwraca wynik (przezroczyste dla klienta; `back-ldap`/`chain` overlay w OpenLDAP).
- **Replikacja** – te same dane na wielu serwerach → dostępność, wydajność odczytu (LDAP jest zoptymalizowany pod odczyty), lokalność.

## Modele replikacji
| Model | Opis | Zalety / wady |
|---|---|---|
| **single-master** (master-slave, _provider-consumer_) | zapisy tylko na masterze; repliki tylko do odczytu; zapis na replice → referral do mastera | brak konfliktów; SPoF dla zapisów |
| **multi-master** (_multi-provider_, N-way) | zapisy na dowolnym serwerze, wzajemna replikacja | dostępność zapisów; możliwe **konflikty** (rozwiązywane np. znacznikiem czasu – ostatni wygrywa) |
| **mirror mode** | dwa mastery replikujące się wzajemnie, ale zapisy kierowane do jednego (np. przez load balancer) | szybki failover bez konfliktów |
| **replikacja częściowa** | tylko wybrane poddrzewo / atrybuty / filtr | oszczędność, lokalne kopie |

Kierunek inicjatywy:
- **push** – dostawca wysyła zmiany do konsumentów,
- **pull** – konsument pobiera zmiany od dostawcy.

## Replikacja w OpenLDAP
### slurpd (historyczne, do 2.3)
- Master `slapd` zapisuje zmiany do **pliku replog**; osobny demon `slurpd` czyta go i **wypycha** (push) zmiany do replik jako operacje LDAP.
- Wady: brak synchronizacji stanu (replika po awarii może się rozjechać), trudna inicjalizacja replik.

### syncrepl (od 2.2, RFC 4533 – LDAP Content Synchronization)
- **Konsument inicjuje** synchronizację (pull) – mechanizm działa w `slapd` konsumenta, dostawca udostępnia overlay `syncprov`.
- Sesja synchronizacji z **cookie** (_contextCSN_ – znacznik zmian) – po restarcie konsument wznawia od ostatniego stanu.
- Tryby:
  - **refreshOnly** – okresowe odpytywanie (`interval=`), dostawca przesyła zmiany od cookie i kończy,
  - **refreshAndPersist** – po początkowej synchronizacji połączenie pozostaje otwarte (**persistent search**), zmiany wysyłane natychmiast.
- Synchronizacja oparta na **stanie** (`entryUUID`, `entryCSN`), a nie na logu → samoczynne odtworzenie spójności, brak konieczności ręcznej inicjalizacji.
- **delta-syncrepl** – zamiast całych wpisów przesyłane są tylko zmienione atrybuty z dziennika `accesslog`, co zmniejsza ruch.
- Multi-master w OpenLDAP (2.4+): syncrepl w obu kierunkach + `olcMirrorMode`/`serverID`.
```ldif
# konsument (cn=config)
olcSyncrepl: rid=001 provider=ldap://master.example.com
  bindmethod=simple binddn="cn=repl,dc=example,dc=com" credentials=secret
  searchbase="dc=example,dc=com" type=refreshAndPersist retry="60 +"
olcUpdateRef: ldap://master.example.com
```
Zob. [[Zarządzanie Systemami Komputerowymi/OpenLDAP]], [[Zarządzanie Systemami Komputerowymi/Syncrepl]].

## Active Directory (Microsoft)
> Usługa katalogowa systemów Windows Server (od 2000). Łączy **katalog zgodny z LDAP (v3)**, **DNS** i uwierzytelnianie **Kerberos**, a do tego centralne zarządzanie kontami, komputerami i politykami w domenie.

### Struktura logiczna
- **Domena** – granica administracyjna i replikacji (np. `firma.local`), zawiera użytkowników, grupy, komputery; nazwa DNS ↔ DN `dc=firma,dc=local`.
- **Jednostka organizacyjna (OU)** – kontener w domenie do grupowania obiektów, delegacji uprawnień i przypinania GPO.
- **Drzewo** – hierarchia domen o **ciągłej przestrzeni nazw** DNS (`firma.local`, `pl.firma.local`).
- **Las** (_forest_) – zbiór drzew współdzielących **schemat**, **konfigurację** i **katalog globalny**; granica bezpieczeństwa.
- **Relacje zaufania** (_trusts_) – automatyczne, **dwukierunkowe i tranzytywne** między domenami lasu; jawne zaufania zewnętrzne/lasów.

### Struktura fizyczna
- **Kontrolery domeny (DC)** – serwery przechowujące replikę katalogu domeny; wszystkie DC są zapisywalne (**multi-master**), z wyjątkiem **RODC** (read-only DC, dla oddziałów).
- **Lokacje** (_sites_) – grupy dobrze połączonych podsieci; sterują wyborem DC przez klientów i harmonogramem replikacji.

### Partycje (konteksty nazw) katalogu
| Partycja | Zawartość | Zasięg replikacji |
|---|---|---|
| **domenowa** | obiekty domeny (użytkownicy, grupy, komputery, OU) | wszystkie DC domeny |
| **konfiguracyjna** | topologia lasu, lokacje, usługi | wszystkie DC lasu |
| **schematu** | definicje klas i atrybutów | wszystkie DC lasu |
| **aplikacyjne** | np. strefy DNS (`DomainDnsZones`) | wybrane DC |

### Replikacja w AD
- **Multi-master**, oparta na stanie i numerach **USN** (_Update Sequence Number_) każdego DC oraz **wektorach aktualności** (_up-to-dateness vector_) – DC wie, które zmiany już zna.
- Granulacja **na poziomie atrybutu**; konflikty rozwiązywane: numer wersji atrybutu → znacznik czasu → GUID DC.
- Topologię tworzy automatycznie **KCC** (_Knowledge Consistency Checker_):
  - **wewnątrz lokacji** – pierścień, powiadomienie o zmianie po kilkunastu sekundach, bez kompresji,
  - **między lokacjami** – przez _bridgehead servers_ wg harmonogramu, z kompresją (RPC/IP).
- Usunięcia jako **tombstone** (usuwane po czasie życia – 180 dni).

### Role FSMO (operacje jednego wzorca)
Niektóre operacje nie mogą być multi-master:
| Rola | Zasięg | Funkcja |
|---|---|---|
| **Schema Master** | las | modyfikacje schematu |
| **Domain Naming Master** | las | dodawanie/usuwanie domen |
| **RID Master** | domena | przydział pul RID (część SID) dla DC |
| **PDC Emulator** | domena | zmiany haseł (priorytet), blokady kont, źródło czasu, zgodność z NT4 |
| **Infrastructure Master** | domena | aktualizacja odwołań do obiektów z innych domen |

### Katalog globalny (Global Catalog)
Częściowa replika (**wybrane atrybuty**) **wszystkich obiektów lasu**, przechowywana na wybranych DC. Pozwala wyszukiwać w całym lesie i jest potrzebny przy logowaniu (członkostwo w grupach uniwersalnych). Porty: LDAP 3268, LDAPS 3269.

### Protokoły i usługi
- **LDAP** (389, LDAPS 636) – dostęp do katalogu; DN w stylu `CN=Jan Kowalski,OU=IT,DC=firma,DC=local`,
- **DNS** z rekordami **SRV** (`_ldap._tcp.dc._msdcs.firma.local`) – lokalizacja DC przez klientów,
- **Kerberos** (88) – domyślne uwierzytelnianie (DC = KDC), NTLM jako starsze – [[25 Mechanizmy egzekwowania polityki uwierzytelniania]],
- **Group Policy Objects (GPO)** – polityki konfiguracji i bezpieczeństwa przypinane do lokacji/domeny/OU (kolejność LSDOU), replikowane przez SYSVOL (DFSR),
- obiekty identyfikowane **GUID** (stały) i **SID** (bezpieczeństwo).

### AD a świat Unix/Linux
- **Samba** jako członek domeny lub **kontroler domeny zgodny z AD** (Samba 4), **Winbind** / **SSSD** do logowania użytkowników AD i SSO – [[Zarządzanie Systemami Komputerowymi/Samba]], [[Zarządzanie Systemami Komputerowymi/Winbind]].
- Odwzorowanie atrybutów POSIX (`uidNumber`, `gidNumber` – RFC 2307 w schemacie AD).

### LDAP (OpenLDAP) vs AD
| | OpenLDAP | Active Directory |
|---|---|---|
| Zakres | ogólna usługa katalogowa | katalog + DNS + Kerberos + GPO + zarządzanie Windows |
| Schemat | dowolny, rozszerzalny | predefiniowany, rozszerzalny (ostrożnie – zmiany w całym lesie) |
| Replikacja | syncrepl (provider/consumer, multi-provider) | multi-master z USN, KCC, FSMO |
| Uwierzytelnianie | bind simple/SASL (Kerberos opcjonalnie) | Kerberos, NTLM |
| Struktura | dowolne DIT | domeny, drzewa, lasy, OU |
