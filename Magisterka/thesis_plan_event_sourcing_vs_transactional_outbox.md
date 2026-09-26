# Plan działania — porównanie Transactional Outbox vs Event Sourcing (EDA, atomowość i wydajność)

Data referencyjna: **2026-04-20**  
Cel: praca magisterska porównująca (głównie wydajnościowo) dwa podejścia do zapewnienia atomowości i publikacji zdarzeń w EDA:
- **Transactional Outbox (TO)**: PostgreSQL + Kafka
- **Event Sourcing (ES)**: KurrentDB/EventStoreDB + Kafka + projekcje w PostgreSQL

Środowisko: całość uruchamialna w **Docker** (lokalnie i w chmurze).  
Load testing: **k6**.  
Metryki/observability: **Micrometer + Prometheus + Grafana**.  
Semantyka publikacji: **at-least-once** + **idempotencja** projekcji/consumerów.

---

## 1. Zakres minimalny (MVP) — zaktualizowany wg wymagań

### 1.1. Transactional Outbox — minimum
- **TO-Z1**: zapis stanu + outbox w transakcji ✅
- **TO-O1**: prosty odczyt z tabeli stanu ✅
- **TO-P1**: polling outbox → Kafka (baseline) ✅
- **TO-P2**: “trigger” jako wariant minimalny ✅  
  **Uwaga implementacyjna:** bezpośrednie publikowanie do Kafka z triggera Postgres jest zwykle niepraktyczne. Przyjmujemy praktyczną interpretację:
  - **TO-P2a (rekomendowane):** Postgres trigger → `NOTIFY` kanałem, a `publisher-to` budzi się i natychmiast pobiera nowe rekordy z outbox (zamiast cyklicznego polling).  
  To zachowuje atomowość (stan+outbox) i redukuje opóźnienie publikacji bez ciągłych zapytań.

### 1.2. Event Sourcing — minimum
- **ES-Z1**: odbudowa stanu z eventów + append do streamu (baseline) ✅
- **ES-Z2**: snapshoty ✅
- **ES-Z3**: cache agregatu ✅
- **ES-O1**: projekcje aktualizowane przy każdym evencie ✅
- **ES-O2**: projekcje aktualizowane cyklicznie / batchowo ✅
- **ES-P1**: natywna subskrypcja KurrentDB → Kafka ✅

### 1.3. Co nie jest w minimum
- TO-P3 (CDC) — opcjonalnie, jeśli starczy czasu
- ES-O3 (leniwe “catch-up on read”) — opcjonalnie

---

## 2. Model domenowy (Inventory)

### 2.1. Agregat / zasób
- **Inventory Item** z `availableQuantity`
- Testy: rezerwowanie `quantity=1` w scenariuszu hot-seat (wielu userów na ten sam item)

### 2.2. Komenda
- `Reserve(itemId, reservationId, quantity)`

### 2.3. Eventy domenowe (minimalnie)
- `InventoryReserved(itemId, reservationId, quantity)`
- Eventów “Rejected” **nie publikujemy** (publikacja dotyczy success-path; odrzucenia mierzymy metrykami API).

---

## 3. Założenia spójności i retry

### 3.1. Spójność odczytu
- Domyślnie **brak read-your-writes** (eventual consistency akceptowalne i mierzone).
- Warianty z cache (ES-Z3) mogą poprawić “perceived read-your-writes” po stronie command-handling, ale nie zmieniamy semantyki odczytów: odczyt ES idzie przez projekcje.

### 3.2. Retry
- Retry **tylko na konflikty**:
  - TO: optimistic lock fail (version mismatch / update 0 rows)
  - ES: wrong expected revision (optimistic concurrency w KurrentDB)
- Parametry retry do ustalenia, propozycja startowa:
  - `maxAttempts=5`
  - backoff: exponential + jitter (`base=5ms`, `cap=100ms`)
- Metryki retry są obowiązkowe (sekcja 10).

---

## 4. Architektura i komponenty (Docker)

### 4.1. Wspólne komponenty
- `kafka`
- `prometheus`, `grafana`

### 4.2. TO stack
- `api-to` (Ktor)
- `publisher-to` (osobny kontener)
- `postgres-to` (może być wspólny Postgres, ale osobny schema dla czystości)

### 4.3. ES stack
- `api-es` (Ktor)
- `publisher-es` (osobny kontener)
- `projection-es` (osobny kontener; consumer group)
- `kurrentdb`
- `postgres-es` (projekcje; może być wspólny Postgres, ale osobny schema)

---

## 5. Warianty TO — szczegóły

### 5.1. TO-Z1 (zapis)
- Transakcja Postgres:
  - update `inventory_state` z optimistic locking
  - insert do `outbox`

### 5.2. TO-O1 (odczyt)
- `GET /inventory/{itemId}` czyta z `inventory_state`

### 5.3. TO-P1 (polling)
- `publisher-to` cyklicznie pobiera rekordy z `outbox` (`FOR UPDATE SKIP LOCKED`) i publikuje do Kafka
- Parametry:
  - `pollIntervalMs`
  - `batchSize`

### 5.4. TO-P2 (trigger → NOTIFY wake-up)
**Cel:** zmniejszyć opóźnienie publikacji i ograniczyć “puste” zapytania polling.

- Trigger w Postgres po INSERT do `outbox`:
  - `NOTIFY outbox_new, '<aggregateId lub eventId>'`
- `publisher-to`:
  - utrzymuje połączenie `LISTEN outbox_new`
  - po odebraniu NOTIFY natychmiast pobiera batch z `outbox` i publikuje do Kafka
- Wciąż używamy `FOR UPDATE SKIP LOCKED` aby wspierać skalowanie horyzontalne publisherów (wiele instancji).

**Metryki porównawcze:**
- TO-P1 vs TO-P2: różnice w `commit→publish`, obciążeniu DB (liczba zapytań), backlog outbox.

---

## 6. Warianty ES — szczegóły

### 6.1. ES-Z1 (zapis bazowy)
- `api-es`:
  1) read stream `inventory-{itemId}`
  2) apply eventy (odbuda stanu)
  3) jeśli rezerwacja możliwa → append event z optimistic concurrency (`expectedRevision`)
  4) retry tylko przy konflikcie

### 6.2. ES-Z2 (snapshoty)
**Cel:** skrócić czas odbudowy agregatu przy długich streamach.

- Snapshot store: Postgres (np. tabela `inventory_snapshots`)
- Polityka:
  - snapshot co `N` eventów (np. 100, 1000) — parametr eksperymentu
- Logika odbudowy:
  - jeśli jest snapshot → start od snapshotu i nakładaj eventy od `snapshotRevision+1`
  - jeśli brak snapshotu → pełny replay

**Metryki:**
- `data_state_fetch_ms{variant=es, method=rebuild|snapshot_partial_rebuild}`
- `events_applied_count`
- `snapshot_hit_total`, `snapshot_miss_total`

**Wymóg testowy:** osobny scenariusz “long-lived aggregate” (sekcja 12.4).

### 6.3. ES-Z3 (cache agregatu)
**Cel:** ograniczyć koszt odczytu streamu przy zapisie oraz wpływ hot-seat na p95/p99.

- Cache po stronie `api-es` (command side), warianty:
  - in-memory (na start)
  - opcjonalnie Redis (jeśli chcesz testować w skali wielu instancji API)
- Cache wpis:
  - klucz: itemId
  - wartość: zmaterializowany stan agregatu + `lastStreamRevision`
- Inwalidacja / odświeżanie:
  - na success append: aktualizuj cache lokalnie
  - na conflict: odśwież z event store (lub drop i rebuild)
  - TTL (parametr)

**Metryki:**
- `aggregate_cache_hit_total`, `aggregate_cache_miss_total`
- wpływ na `data_state_fetch_ms{method=cache_hit}`, `reserve_command_seconds` i `reserve_conflicts_total`

### 6.4. ES-P1 (publikacja)
- `publisher-es` subskrybuje KurrentDB i publikuje eventy domenowe do Kafka (key=itemId)

### 6.5. ES-O1 (projekcja per-event)
- `projection-es` konsumuje eventy z Kafka i aktualizuje `inventory_projection` per event (niemal natychmiast)

### 6.6. ES-O2 (projekcja cykliczna / batch)
**Cel:** sprawdzić trade-off: większy commit→read vs mniejsze koszty stałego update.

Implementacja rekomendowana (O2a):
- `projection-es` buforuje eventy w pamięci
- co `batchIntervalMs` lub po `batchSize` wykonuje batch update w Postgres
- nadal obowiązuje ordering per itemId i idempotencja

Metryki:
- `es_commit_to_project_ms` (powinno wzrosnąć)
- CPU/IO DB vs O1
- throughput projekcji

---

## 7. Kafka: ordering, envelope, DLQ (ES)

### 7.1. Ordering per itemId
- key = `itemId` (aggregateId)
- kolejność gwarantowana w obrębie partycji

### 7.2. ES event envelope (JSON)
Topic: `es.inventory-events`  
Key: itemId

Value:
```json
{
  "eventId": "uuid",
  "aggregateId": "itemId",
  "streamName": "inventory-{itemId}",
  "streamRevision": 123,
  "committedAt": "2026-04-20T12:34:56.789Z",
  "type": "InventoryReserved",
  "data": {
    "reservationId": "uuid",
    "quantity": 1
  }
}
```

### 7.3. DLQ
Topic: `es.inventory-events.dlq`  
Powody: `gapTimeout`, `bufferOverflow`, `deserializationError`

---

## 8. Idempotencja projekcji ES + obsługa gap

### 8.1. Checkpoint w projekcji
W `inventory_projection` trzymamy:
- `last_stream_revision`

Reguły:
- `rev <= last` → skip (duplikat/replay)
- `rev == last+1` → apply
- `rev > last+1` → gap

### 8.2. GAP: buffer + TTL + DLQ (degradacja kontrolowana)
Parametry (start):
- `pendingTtl=30s`
- `pendingMaxPerItem=100`
- `pendingTotalMax=100000`

Mechanizm:
- gap → event trafia do `pending`
- housekeeping próbuje “drain” jeśli brakujące revision dotrą
- po TTL → DLQ + metryki (`projection_gap_dlq_total`)

---

## 9. Storage: minimalne schematy

### 9.1. Postgres (TO)
- `inventory_state(item_id PK, available_qty, version, updated_at)`
- `outbox(id PK, aggregate_id, event_id, event_type, payload_json, created_at, published_at NULL, status, attempt_count, last_error NULL)`

### 9.2. Postgres (ES projekcje)
`inventory_projection`:
- `item_id` PK
- `available_qty` int NOT NULL
- `last_stream_revision` bigint NOT NULL
- `last_event_id` uuid NOT NULL
- `last_committed_at` timestamptz NOT NULL
- `projected_at` timestamptz NOT NULL

### 9.3. Postgres (ES snapshoty)
Przykładowo:
- `inventory_snapshots(item_id PK, snapshot_revision bigint, snapshot_json jsonb, created_at timestamptz)`

---

## 10. Metryki (Micrometer/Prometheus)

> **Nota o zmianach (2026-05-05)**
>
> **Zmiana nazwy:** `aggregate_rebuild_ms` → `data_state_fetch_ms{variant, method}` (wspólna metryka dla TO i ES).
> Dla TO mierzy czas prostego zapytania do `inventory_state`. Dla ES mierzy czas pełnego rebuild, partial rebuild (od snapshotu) lub cache hit.
> Ujednolicona nazwa umożliwia porównanie obu wariantów na jednym wykresie Grafana, a label `method` zachowuje semantykę.
>
> **Potencjalnie redundantne:** `projection_events_applied_total` i `to_db_queries_total` — oznaczone poniżej.

### 10.1. API (TO i ES)
- latency HTTP: p50/p95/p99 (`http_server_requests_seconds`)
- `reserve_command_seconds{result=success|rejected|conflict_retried|conflict_failed}`
- `reserve_conflicts_total`
- `reserve_retries_total`

### 10.2. TO outbox/publish
- `outbox_pending_count` (gauge)
- `to_commit_to_publish_ms` (histogram)
- `to_db_queries_total` ⚠️ **potencjalnie redundantne** — częściowo zastępowane przez `to_notify_wakeups_total` w scenariuszu P2; dodać tylko jeśli potrzebne do diagnozy liczby zapytań przy porównaniu P1 vs P2
- `to_notify_wakeups_total` (dla P2)

### 10.3. Wspólne: pobieranie stanu agregatu (TO i ES)
- `data_state_fetch_ms{variant=to|es, method=query|rebuild|snapshot_partial_rebuild|cache_hit}` (histogram)
  — zastępuje `aggregate_rebuild_ms`; dla TO `method=query` zawsze; dla ES zmienia się wraz z wariantami Z1/Z2/Z3
  — umożliwia bezpośrednie porównanie kosztu pobierania stanu między TO a ES na jednym panelu

### 10.4. ES write/rebuild/snapshot/cache
- `events_applied_count` (histogram) — liczba eventów nałożonych przy rebuild; kluczowa zmienna objaśniająca wzrost `data_state_fetch_ms` przy długich streamach; niezbędna dla eksperymentu 12.4
- `snapshot_hit_total`, `snapshot_miss_total`
- `aggregate_cache_hit_total`, `aggregate_cache_miss_total`

### 10.5. ES publish/projection
- `es_commit_to_publish_ms` (histogram)
- `es_commit_to_project_ms` (histogram)
- `projection_events_applied_total` ⚠️ **potencjalnie redundantne** — throughput projekcji jest w dużej mierze pokryty przez Kafka consumer lag; przydatne tylko do wykrywania cichego pomijania eventów (rozbieżność między liczbą opublikowanych a zastosowanych); dodać jeśli zajdzie potrzeba diagnozy
- `projection_duplicates_total`
- `projection_gaps_detected_total`
- `projection_gap_dlq_total`
- `projection_pending_total` (gauge)

Kafka:
- consumer lag dla `projection-es` (eksporter/JMX)

---

## 11. Implementacja — kolejność prac (task list)

### Etap 0 (2–3 dni): Infra + repo
- docker-compose z profilami: `to`, `es`, `obs`
- Makefile/skrypty: `make up-to`, `make up-es`, `make up-obs`
- podstawowe dashboardy Grafana + Prometheus scrape

### Etap 1 (1 tydzień): Wspólne fundamenty
- wspólne DTO i kontrakt event envelope
- k6: hot-seat + low contention + 80/20
- Micrometer: podstawowe timery/liczniki

### Etap 2 (1 tydzień): TO baseline (P1) + metryki
- TO-Z1, TO-O1, TO-P1
- skalowanie publisher-to (min. 1 i 2 instancje)
- zapis wyników testów

### Etap 3 (3–5 dni): TO-P2 (trigger/notify)
- trigger + NOTIFY
- publisher-to LISTEN + wake-up
- porównanie P1 vs P2 (opóźnienia + liczba zapytań)

### Etap 4 (1–1.5 tygodnia): ES baseline (Z1, O1, P1) w wersji S2
- api-es: rebuild + append + retry
- publisher-es: subskrypcja KurrentDB → Kafka
- projection-es: per-event projekcja (O1) + idempotencja revision
- DLQ + metryki

### Etap 5 (1 tydzień): ES-Z2 (snapshoty) + test long-lived
- snapshot store + polityka N
- scenariusz testowy z długim streamem (sekcja 12.4)
- wykresy: rebuild time vs liczba eventów

### Etap 6 (1 tydzień): ES-Z3 (cache) + eksperymenty hot-seat
- cache agregatu
- wpływ na latency, retry, conflicts

### Etap 7 (1 tydzień): ES-O2 (batch projekcje)
- batchIntervalMs/batchSize
- porównanie O1 vs O2: commit→project + obciążenie DB + throughput

### Etap 8 (ciągle): automatyzacja uruchomień i zbierania wyników
- standard run: warmup + measure + powtórzenia
- export wyników k6 i metryk Prometheus do katalogu `results/`

---

## 12. Plan eksperymentów (minimalny) — rozszerzony pod Z2/Z3/O2

### 12.1. Stałe (propozycja startowa)
- `initialQuantity=1000`, `quantity=1`
- warmup 2 min + measure 10 min
- powtórzenia: 5

### 12.2. Scenariusze ruchu (k6)
- **E1 Low contention**: wiele itemId, rozkład równy
- **E2 Hot-seat**: 1 itemId
- **E3 80/20**: 80% na 1 itemId, 20% rozproszone

### 12.3. Skale
- VU: 10, 50, 200, 500 (dopasować do zasobów)
- TO: publisher-to instancje: 1 / 2 / 4
- ES: partycje topicu i konsumenci projekcji: 1/3/6 (do ustalenia)

### 12.4. Test “long-lived aggregate” (wymagany dla ES-Z2)
- jeden `itemId` dostaje dużo eventów (np. 1k, 10k)
- mierz:
  - `data_state_fetch_ms{variant=es, method=rebuild}` vs liczba eventów
  - `events_applied_count` vs liczba eventów (korelacja)
  - snapshot frequency (N=100, 500, 1000) — sweep; efekt widoczny przez zmianę `method` z `rebuild` na `snapshot_partial_rebuild`

---

## 13. Eksperymenty awaryjne (niezawodność)
- F1: restart `projection-es` w trakcie obciążenia → duplicates + lag
- F2: restart `publisher-es` w trakcie obciążenia → commit→publish + lag
- (opcjonalnie) F3: kontrolowany błąd keyingu → wzrost gaps/out-of-order

---

## 14. Harmonogram (do oddania)
- **Maj 2026**: Etap 0–3 (TO + TO-P2)
- **Czerwiec 2026**: Etap 4 (ES baseline)
- **Lipiec 2026**: Etap 5–7 (snapshoty, cache, O2) + eksperymenty
- **Sierpień 2026**: finalne runy, wykresy, wnioski, pisanie
- **Wrzesień 2026**: oddanie pracy

---

## 15. Otwarte decyzje do dopięcia
1) Dokładne parametry retry (`maxAttempts`, backoff)
2) Parametry TO-P1 (pollIntervalMs, batchSize)
3) Parametry TO-P2 (czy NOTIFY per event czy per transakcja/batch)
4) Parametry ES snapshotów (N) i cache (TTL/rozmiar, in-memory vs Redis)
5) Parametry ES-O2 (batchIntervalMs, batchSize)
6) Liczba partycji w topicu `es.inventory-events` (baseline i skala)




> ## 1. Odczyt  
> ---  

> ### Metryki  
> - Ile trwają zapytania  
> - Po jakim czasie zmiany są widoczne w zapytaniach  
> - Jakie zużycie zasobów przy zapytaniach/utrzymywaniu projekcji  
> - Przepustowość (zapytań/s)  
> - Jak różne optymalizacje wpływają na metryki  
> - Ile czasu trwają zapisy  
>         - z różnym natężeniem konfliktów  
> - Przepustowość (zapisów/s)  
> - Jakie zużycie zasobów  
> - Jak różne optymalizacje wpływają na metryki  
> - Jakie opóźnienie występuję pomiędzy zapisem, a opublikowaniem zdarzenia  
> - Przepustowość (eventów/s) — czy wąskie gardło jest w zapisie czy w publikacji  
> - Consumer lag — ile eventów czeka na opublikowanie w danej chwili, szczególnie przy skokach obciążenia  
> - Jakie zużycie zasobów