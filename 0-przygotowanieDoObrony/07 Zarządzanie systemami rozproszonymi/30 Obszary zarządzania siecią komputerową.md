---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 30
---
# 30. Obszary zarządzania siecią komputerową
---
> **Zarządzanie siecią** to działania, metody i narzędzia służące do **eksploatacji, administrowania, utrzymania i konfigurowania** sieci i systemów sieciowych. Celem jest zapewnienie wymaganej **dostępności, wydajności, bezpieczeństwa** i **kosztu** usług. Standardowy podział na obszary to model **FCAPS**, zdefiniowany przez ISO w ramach zarządzania OSI (ISO/IEC 7498-4, ISO 10040) i przyjęty przez ITU-T w modelu TMN (M.3400).

## Model FCAPS – pięć obszarów funkcjonalnych
| Litera | Obszar | Cel |
|---|---|---|
| **F** | zarządzanie **uszkodzeniami** (_Fault_) | wykrywanie, lokalizacja, izolacja i usuwanie awarii |
| **C** | zarządzanie **konfiguracją** (_Configuration_) | inwentaryzacja, konfiguracja i kontrola zmian elementów sieci |
| **A** | zarządzanie **rozliczeniami** (_Accounting_) | pomiar wykorzystania zasobów, naliczanie kosztów, limity |
| **P** | zarządzanie **wydajnością** (_Performance_) | monitorowanie i optymalizacja parametrów pracy sieci |
| **S** | zarządzanie **bezpieczeństwem** (_Security_) | ochrona sieci i informacji zarządczych przed nieuprawnionym dostępem |

### F – zarządzanie uszkodzeniami (awariami)
- **Wykrywanie** awarii: alarmy i zdarzenia od urządzeń (SNMP **trap**/**inform**, syslog), aktywne sprawdzanie dostępności (ping, sondy usług), progi metryk.
- **Rejestracja** zdarzeń i alarmów (dziennik), **korelacja** – ustalenie przyczyny źródłowej, gdy jedna awaria (np. przełącznik szkieletowy) generuje setki alarmów; tłumienie alarmów wtórnych.
- **Diagnostyka i lokalizacja**: testy (`ping`, `traceroute`, testy pętli, analiza ruchu), izolacja uszkodzonego elementu.
- **Usuwanie**: rekonfiguracja, przełączenie na zapas (redundancja, protokoły failover), naprawa, wymiana.
- **Obsługa zgłoszeń** (_trouble ticketing_), eskalacja, powiadomienia (e-mail, SMS, pager).
- **Miary**: dostępność, MTBF (średni czas między awariami), MTTR (średni czas naprawy), liczba awarii.
- Proaktywnie: przewidywanie awarii na podstawie trendów (błędy CRC, temperatura).

### C – zarządzanie konfiguracją
- **Inwentaryzacja** – baza elementów sieci (CMDB): urządzenia, moduły, wersje oprogramowania, numery seryjne, licencje; **odkrywanie topologii** (LLDP/CDP, skanowanie, tablice ARP/MAC).
- **Konfigurowanie** elementów: parametry interfejsów, VLAN, routing, adresacja IP (IPAM, DHCP), usługi.
- **Kontrola zmian**: wersjonowanie i **kopie zapasowe konfiguracji**, porównywanie z konfiguracją wzorcową (wykrywanie dryfu), zatwierdzanie i wycofywanie zmian, okna serwisowe.
- **Zarządzanie oprogramowaniem** urządzeń (aktualizacje firmware'u).
- **Automatyzacja**: szablony, NETCONF/YANG, RESTCONF, Ansible, konfiguracja jako kod – [[Zarządzanie Systemami Rozproszonymi/Configuration Management as Code]], [[Zarządzanie Systemami Rozproszonymi/Ansible]], [[Zarządzanie Systemami Rozproszonymi/IaC]].
- Konfiguracja przez SNMP `SET` (w praktyce rzadko – ograniczone możliwości, słabe bezpieczeństwo v1/v2c).

### A – zarządzanie rozliczeniami
- **Pomiar wykorzystania** zasobów przez użytkowników, działy, klientów: ilość przesłanych danych, czas połączeń, wykorzystanie łączy i usług.
- Źródła danych: **NetFlow / IPFIX / sFlow** (rekordy przepływów: adresy, porty, bajty, pakiety, czas), liczniki interfejsów SNMP, RADIUS **Accounting** (start/stop sesji – [[Bezpieczeństwo Systemów Rozproszonych/Radius]]).
- **Naliczanie opłat** (_billing_) i rozliczenia wewnętrzne (_chargeback_).
- **Limity i kwoty** (przydziały pasma, transferu), egzekwowanie polityk wykorzystania.
- **Planowanie pojemności** – uzasadnienie inwestycji na podstawie danych o wykorzystaniu.
- Wykrywanie nadużyć (nietypowo duży ruch).

### P – zarządzanie wydajnością
- **Monitorowanie parametrów**: przepustowość i wykorzystanie łączy, **opóźnienie**, **zmienność opóźnienia (jitter)**, **straty pakietów**, błędy, obciążenie CPU/pamięci urządzeń, czasy odpowiedzi usług.
- **Zbieranie danych** okresowo (odpytywanie SNMP, telemetria strumieniowa – gNMI), przechowywanie jako szeregi czasowe, **analiza trendów**.
- **Wartości progowe** i alarmy o degradacji (połączenie z F).
- **Jakość usług (QoS)** – klasyfikacja i priorytetyzacja ruchu, weryfikacja **SLA** (umów o poziomie usług).
- **Optymalizacja**: zmiany routingu, równoważenie obciążenia, rozbudowa łączy, **planowanie pojemności** (_capacity planning_).
- Narzędzia: RMON ([[32 Baza informacji zarządzania MIB#RMON]]), MRTG/Cacti, Zabbix, [[Zarządzanie Systemami Rozproszonymi/Prometheus]] + [[Zarządzanie Systemami Rozproszonymi/Grafana]], IP SLA, iperf.

### S – zarządzanie bezpieczeństwem
- **Kontrola dostępu** do sieci i do **samych funkcji zarządzania** (kto może konfigurować urządzenia): AAA (RADIUS/TACACS+), 802.1X, ACL na interfejsach zarządzających, osobna sieć zarządzania (_out-of-band_).
- Zarządzanie **kluczami, certyfikatami i hasłami** urządzeń; SNMPv3 z uwierzytelnianiem i szyfrowaniem ([[31 Protokół SNMP#SNMPv3 – bezpieczeństwo]]).
- **Monitorowanie zdarzeń bezpieczeństwa**: firewalle, IDS/IPS, logi (SIEM), wykrywanie anomalii, audyt.
- Polityki bezpieczeństwa, segmentacja (VLAN, strefy), zarządzanie podatnościami i aktualizacjami, reagowanie na incydenty.
- Ochrona **informacji zarządczej** (MIB, konfiguracje, logi) przed odczytem i modyfikacją.
- Powiązanie z systemem zarządzania bezpieczeństwem informacji – [[Zarządzanie Systemami Rozproszonymi/ISO 27001]].

## Architektura systemu zarządzania
### Model zarządca–agent
```
┌────────────────────────┐        protokół zarządzania        ┌──────────────────────────┐
│  Stacja zarządzająca    │  ─── żądania (GET/SET) ────────▶   │  Element sieci (router)  │
│  NMS: zarządca          │  ◀── odpowiedzi / powiadomienia ── │  agent + baza MIB        │
│  aplikacje FCAPS, baza  │        (trap / inform)             │  obiekty zarządzane      │
└────────────────────────┘                                     └──────────────────────────┘
```
- **Zarządca** (_manager_, NMS) – aplikacja odpytująca agentów, zbierająca dane, prezentująca je operatorom, wysyłająca polecenia.
- **Agent** – oprogramowanie w urządzeniu, udostępnia informacje i realizuje polecenia.
- **Obiekt zarządzany** – abstrakcyjna reprezentacja zasobu (interfejs, licznik, tablica routingu).
- **Baza informacji zarządzania (MIB)** – ustrukturyzowany zbiór obiektów zarządzanych – [[32 Baza informacji zarządzania MIB]].
- **Protokół zarządzania** – SNMP ([[31 Protokół SNMP]]), CMIP (OSI), NETCONF, gNMI.
- **Pośrednik** (_proxy agent_) – dla urządzeń, które nie obsługują protokołu.
- **Odpytywanie** (_polling_) vs **powiadomienia** (_trap-directed polling_: zarządca rzadko odpytuje, a na trap reaguje szczegółowym odpytaniem).

### Warianty organizacji
- **scentralizowane** – jedna stacja NMS (prosto, SPoF, skalowalność),
- **hierarchiczne** – zarządcy lokalni raportują do centralnego (duże sieci operatorskie),
- **rozproszone** – współpracujący zarządcy, delegacja zadań.
- **In-band** (zarządzanie przez tę samą sieć, co ruch użytkowników) vs **out-of-band** (osobna sieć/port konsoli – dostęp mimo awarii sieci produkcyjnej).

### Model TMN (ITU-T M.3010) – warstwy zarządzania (operatorzy telekomunikacyjni)
1. **zarządzanie elementami** (_Element Management_) – pojedyncze urządzenia,
2. **zarządzanie siecią** (_Network Management_) – sieć jako całość, topologia,
3. **zarządzanie usługami** (_Service Management_) – usługi dla klientów, SLA,
4. **zarządzanie biznesowe** (_Business Management_) – cele przedsiębiorstwa.

Każda warstwa realizuje funkcje FCAPS. Nowsze ramy: **eTOM** (TM Forum), **ITIL** (procesy zarządzania usługami IT: incydenty ≈ F, zmiany i konfiguracja ≈ C, pojemność i dostępność ≈ P).

## Współczesne narzędzia i trendy
- **NMS**: Zabbix, Nagios/Icinga, LibreNMS, PRTG, SolarWinds, OpenNMS,
- **monitorowanie usług i aplikacji**: Prometheus/Grafana, ELK – [[Monitorowanie w Systemach Rozproszonych]],
- **telemetria strumieniowa** (push, gNMI, OpenConfig) zamiast odpytywania SNMP,
- **SDN** – centralny kontroler zarządza płaszczyzną sterowania (OpenFlow),
- **automatyzacja i intent-based networking**, AIOps (korelacja zdarzeń przez ML).
