# Różnice
## 1. Odczyt
---
#### ES
##### Zapytania
- Są wykonywane na projekcjach, eventual consistency
- Mogą być różne sposoby na aktualizacje projekcji: (do zagłębienia)
	- każdy event jest łapany aby aktualizować projekcje
	- co jakiś czas
	- co zapytanie pobierane są event-y które nie zostały nałożone na projekcje i jest aktualizowana przed zwróceniem wyniku
##### Do zapisu
- Stan jest budowany na podstawie event-ów danego agregatu, poprzez nakładanie po kolei wszystkich event-ów
- Możliwe optymalizacje:
	- Snapshot-y - konieczne przy długo żyjących (zawierających dużo event-ów) agregatów
	- Cache
#### TO
- Proste zapytanie do bazy danych, stan danych w bazie (serwisu wprowadzającego zmianę) jest zawsze aktualny
### Metryki
- Ile trwają zapytania
- Po jakim czasie zmiany są widoczne w zapytaniach
- Jakie zużycie zasobów przy zapytaniach/utrzymywaniu projekcji
- Przepustowość (zapytań/s)
- Jak różne optymalizacje wpływają na metryki
## 2. Zapis
#### ES
- Odtworzenie stanu agregatu, przy możliwej zmianie dopisanie event-u do odpowiedniego stream-u
#### TO
- Zapis stanu, jest wykonywany w transakcji z powstałym event-em
### Metryki
- Ile czasu trwają zapisy
	- z różnym natężeniem konfliktów
- Przepustowość (zapisów/s)
- Jakie zużycie zasobów
- Jak różne optymalizacje wpływają na metryki

## 3. Publikacja Event-ów
#### ES
- Przy wykorzystaniu event-store jak KurrentDB event jest publikowany po jego zapisie (pull/subscription), w przypadku nie wyspecjalizowanej bazy danych, konieczna może być implementacja procesu, zapinający się na zmianę w strumieniu lub poll-ujący.
#### TO
- Jakiś inny proces pobiera zapisane i nieopublikowane event-y. Różne strategie zależne od możliwości bazy danych: (do zagłębienia)
	- Trigger-y bazy danych gdy coś się zmieni
	- Poll-owanie co jakiś czas czy nie ma nowych nieopublikowanych zdarzeń
	- Zewnętrzny proces monitoruje log bazy danych (CDC) i przy wpisie do outbox publikuje event
* W teorii możliwy jest wariant bez tabeli outbox gdzie zmiany w bazie (łapane poprzez trigger lub CDC) tłumaczą zmianę na event domenowy
### Metryki
- Jakie opóźnienie występuję pomiędzy zapisem, a opublikowaniem zdarzenia
- Przepustowość (eventów/s) — czy wąskie gardło jest w zapisie czy w publikacji
- Consumer lag — ile eventów czeka na opublikowanie w danej chwili, szczególnie przy skokach obciążenia 
- Jakie zużycie zasobów'

# Implementacja

## Transactional Outbox

### Zapis
- **TO-Z1**: Zapis stanu + outbox w transakcji (wariant bazowy)

### Odczyt
- **TO-O1**: Proste zapytanie do tabeli stanu

### Publikacja
- **TO-P1**: Polling — cykliczne sprawdzanie nieopublikowanych eventów
- **TO-P2**: Trigger bazodanowy wyzwalający publikację
- **TO-P3**: CDC (Change Data Capture) monitorujący log bazy
---

## Event Sourcing

### Zapis
- **ES-Z1**: Odtworzenie stanu agregatu z event-ów + dopisanie nowego eventu do streamu (bez optymalizacji)
- **ES-Z2**: Odtworzenie stanu z wykorzystaniem snapshot-ów
- **ES-Z3**: Odtworzenie stanu z wykorzystaniem cache

### Odczyt (zapytania — projekcje)
- **ES-O1**: Projekcje aktualizowane przy każdym evencie
- **ES-O2**: Projekcje aktualizowane cyklicznie
- **ES-O3**: Projekcje aktualizowane leniwie (przed zapytaniem nakładane nowe eventy)

### Publikacja
- **ES-P1**: KurrentDB (wyspecjalizowany event store) — natywna subskrypcja po zapisie eventu
- **ES-P2**: Polling na strumieniu eventów (niewyspecjalizowana baza)
- **ES-P3**: Subskrypcja na zmiany w strumieniu (np. listen/notify)




---
- Minimalne wycinki:
	- Tylko rezerwowanie zasobów
- Jako serwis aby można "uderzać" z zewnątrz (ktor - mniejszy od spring-a)
- Micrometer do używanych zasobów
- K6 do testów i metryk requestów
