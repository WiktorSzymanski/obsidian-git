---
tags:
  - obrona
up: "[[NPR 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 8
source: "slajdy–merged.pdf"
slajdy: "1–9"
---
# NPR 08. MOM i systemy kolejkowania komunikatów
---
> Wprowadzenie do **oprogramowania pośredniczącego zorientowanego na komunikaty** (_message-oriented middleware_, MOM) — podejścia przeciwstawnego wobec [[NPR 01 Zdalne wywoływanie procedur (RPC)|RPC]] i [[NPR 04 Podejście obiektowe - Java RMI|RMI]]. Zamiast ukrywać sieć pod składnią wywołania, MOM **jawnie wprowadza pośrednika** — kolejkę — i dzięki temu rozłącza nadawcę i odbiorcę **w czasie i w przestrzeni**. Wykład omawia dwa paradygmaty (**kolejkowanie punkt-punkt** i **publish/subscribe**), model pojęciowy systemu i listę realizacji. Szczegóły dwóch z nich — w [[NPR 09 ZeroMQ]] i [[NPR 10 JMS]].

---
## Plan wykładu
<sub>slajdy–merged.pdf, slajd 2</sub>

1. **Koncepcja** — paradygmat kolejkowania (punkt-punkt), paradygmat publish/subscribe
2. **Model systemu**
3. **Przykłady rozwiązań**
4. **JMS**

---
## Cechy MOM
<sub>slajdy–merged.pdf, slajd 3</sub>

- **Uniezależnienie funkcjonowania składników aplikacji od dostępności informacji o interfejsach innych składników.**
- **Uniezależnienie funkcjonowania warstwy komunikacyjnej** (kanału komunikacyjnego) **od działania (obecności) komunikujących się procesów** → **komunikacja nieustanna**.
- **Mechanizm komunikacji pośredniej**:
  - oparty na identyfikacji **miejsca pośredniczącego** (tzw. skrzynki), **a nie adresu nadawcy/odbiorcy**,
  - komunikujące się strony **nie muszą znać się wzajemnie**.
- **Łatwość wdrożenia komunikacji asynchronicznej.**

Te cztery punkty można czytać jako **systematyczne zaprzeczenie założeń RPC**. RPC wymaga, żeby klient znał interfejs serwera (tu: nie wymaga), żeby serwer działał w chwili wywołania (tu: nie musi), żeby klient znał adres serwera (tu: zna tylko nazwę kolejki) i domyślnie blokuje klienta na czas wywołania (tu: nie blokuje).

**Komunikacja nieustanna** (_persistent communication_) to własność, w której wiadomość jest przechowywana w celu doręczenia do odbiorcy **nawet gdy odbiorca nie działa** w danej chwili, a **nadawca zakończył działanie** po jej wysłaniu — w przeciwieństwie do **komunikacji przejściowej**, w której wiadomość jest utrzymywana pod warunkiem, że działają obie strony. Por. [[Narzędzia Przetwarzania Rozproszonego/Trwałość Komunikacji]] i [[Narzędzia Przetwarzania Rozproszonego/Synchroniczność komunikacji]].

---
## Paradygmat kolejkowania (punkt-punkt)
<sub>slajdy–merged.pdf, slajd 4</sub>

- **Komunikacja punkt-punkt.**
- **Organizacja warstwy komunikacyjnej w postaci systemu kolejek** (ang. _queue_) — **{skrzynka ≡ kolejka}** — realizowanych w oparciu o zasoby pamięci, **w tym pamięci dyskowej** (gwarancja trwałości na wypadek awarii → **niezawodność**).
- Udostępnienie mechanizmów komunikacji polegających na:
  - **umieszczeniu** komunikatów w kolejkach,
  - **pobieraniu** komunikatów z kolejek.

Cały interfejs sprowadza się więc do **dwóch operacji** — to najprostszy możliwy model. Wzmianka o pamięci dyskowej nie jest szczegółem implementacyjnym: **to z niej bierze się niezawodność** całego podejścia, bo kolejka przetrwa awarię brokera.

---
## Paradygmat publish/subscribe
<sub>slajdy–merged.pdf, slajd 5</sub>

- **Komunikacja jeden do wielu** (potencjalnie **wielu do wielu**).
- **Strona publikująca (nadawca) udostępnia treść związaną z określonym tematem** (ang. _topic_) — **{skrzynka ≡ temat}**.
- **Środowisko komunikacyjne (usługa) przekazuje treść udostępnionych wiadomości odbiorcom (subskrybentom), którzy zarejestrowali (zapisali) się wcześniej na dany temat.**

Zwróć uwagę na słowo **„wcześniej"**: subskrypcja musi poprzedzać publikację, inaczej wiadomość do subskrybenta nie trafi. To zasadnicza różnica wobec kolejki, w której wiadomość **czeka** na odbiorcę. JMS łagodzi ją mechanizmem **subskrypcji trwałych** — zob. [[NPR 10 JMS#Subskrypcje trwałe|NPR 10]].

---
## Model systemu typu MOM
<sub>slajdy–merged.pdf, slajd 6</sub>

| Pojęcie | Definicja |
|---|---|
| **Wiadomość** (ang. _message_) | **porcja danych** (często z dodatkowymi własnościami) składowana w kolejce |
| **Kolejka** (ang. _queue_) | **miejsce przechowywania** wiadomości (komunikatów); **{temat ≅ kolejka dla wielu odbiorców**, ang. _multiconsumer queue_**}** |
| **Proces** (ang. _process_) | **element aplikacji**, zlecający operacje dotyczące wiadomości w kolejce |
| **Zarządca zbioru kolejek** | **moduł** (proces na określonym węźle, **dostawca, broker**), odpowiedzialny za wykonywanie operacji na kolejkach: tworzenie, usuwanie, lokalizowanie kolejek, definiowanie atrybutów kolejek itp. |

Definicja kolejki zawiera ważne uproszczenie pojęciowe: **temat to po prostu kolejka dla wielu odbiorców**. Oba paradygmaty da się więc opisać jednym modelem — różnią się liczbą konsumentów przypadających na skrzynkę, a nie naturą.

### Funkcjonowanie systemu kolejkowania
<sub>slajd 7</sub>

![[npr-mom-s07-funkcjonowanie-systemu.png]]
<sub>Procesy komunikują się przez router (brokera) z kolejkami opartymi o dysk. slajdy–merged.pdf, slajd 7</sub>

### Warianty komunikacji
<sub>slajd 8</sub>

![[npr-mom-s08-warianty-komunikacji.png]]
<sub>Cztery warianty; **szare zamalowanie oznacza aktywność procesu**, białe — jej brak. slajdy–merged.pdf, slajd 8</sub>

> [!note] Uzupełnienie z Opracowania
> Slajd nie wyjaśnia konwencji ani sytuacji. Opracowanie: szare zamalowanie = **aktywność procesu**, białe = **brak aktywności**. Obrazki 1 i 2 pokazują jedną sekwencję — (1) proces górny wysyła komunikat, **proces dolny jest nieaktywny**; (2) proces dolny **uaktywnia się i pobiera** komunikat. Obrazki 3 i 4 pokazują drugą — (3) **oba procesy są aktywne**, wiadomość została umieszczona w kolejce; (4) proces drugi **przed otrzymaniem wiadomości został zdezaktywowany**, a komunikat czeka. <sub>Opracowanie.pdf, s. 47</sub>

Diagram jest więc ilustracją **komunikacji nieustannej** ze slajdu 3: pokazuje, że nadawca i odbiorca **nigdy nie muszą działać jednocześnie**.

---
## Przykłady rozwiązań
<sub>slajdy–merged.pdf, slajd 9</sub>

- **WebSphere MQ** (IBM MQSeries, **XMS** — _Message Service Client_)
- **Microsoft Message Queuing** (MSMQ)
- **Java Message Service (JMS)** — **standard, specyfikacja interfejsu**. Implementacje oparte na JMS:
  - Sun Java System Message Queue (**OpenMQ** — wersja open source),
  - **Apache ActiveMQ**
- **Oracle Advanced Queueing**
- **RabbitMQ**
- **ZeroMQ**
- **Apache Kafka**

Na liście warto rozróżnić trzy rzeczy: **JMS jest specyfikacją**, nie produktem (stąd osobna lista implementacji); **ZeroMQ nie ma brokera**, więc odstaje od modelu ze slajdu 6; **Kafka** to log z trwałą historią, a nie kolejka, z której wiadomość znika po odebraniu.

---
## Braki w prezentacjach

> [!todo] Czego nie ma na slajdach 1–9
> - **Porównanie MOM z RPC** — mimo że slajd 3 jest w istocie listą różnic, nigdzie nie postawiono ich obok siebie. Zestawienie znajduje się w [[NPR 08 Podejścia do budowy systemów rozproszonych|skompresowane/NPR 08]].
> - **Sprzężenie w czasie i przestrzeni** (_space/time decoupling_) — nazwane pojęcie, którym opisuje się to, co slajd 3 mówi opisowo, nie pada.
> - **Trasowanie i topologie brokerów**, kolejki rozproszone, mostki między brokerami — slajd 7 pokazuje jeden router bez omówienia.
> - **Gwarancje dostarczenia** (_at-most-once_, _at-least-once_, _exactly-once_) w kontekście MOM — pojęcia z [[NPR 01 Zdalne wywoływanie procedur (RPC)#Gwarancja wykonania — semantyka błędu|NPR 01]] nie są tu przywołane; częściowo wraca to przy trybach potwierdzeń JMS.
> - **Kafka** jest wymieniona na liście, ale jej model (log, offsety, grupy konsumentów) nie jest omówiony ani porównany z klasyczną kolejką.
