---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 13
---
# 13. Usługi sieciowe REST
---
> **REST** (_Representational State Transfer_, Roy Fielding, 2000) to **styl architektoniczny** systemów hipermedialnych, a nie protokół. Usługa REST udostępnia **zasoby** identyfikowane przez **URI**. Klient manipuluje ich **reprezentacjami** za pomocą **jednolitego interfejsu** protokołu HTTP (GET, PUT, POST, DELETE…).

## Ograniczenia architektury REST (Fielding)
1. **Klient-serwer** – rozdzielenie interfejsu użytkownika od składowania danych; niezależna ewolucja.
2. **Bezstanowość** – każde żądanie zawiera komplet informacji potrzebnych do jego obsługi; serwer nie przechowuje stanu sesji klienta. Daje skalowalność, niezawodność i widoczność, kosztem narzutu danych w żądaniach.
3. **Pamięć podręczna** – odpowiedzi jawnie oznaczone jako cache'owalne lub nie (`Cache-Control`, `ETag`, `Last-Modified`).
4. **Jednolity interfejs**:
   - identyfikacja zasobów (URI),
   - manipulacja zasobami przez reprezentacje,
   - samoopisujące się komunikaty (typ mediów, metoda, kody statusu),
   - **hipermedia jako silnik stanu aplikacji** (HATEOAS) – [[14 Architektura zorientowana na zasoby - ROA, HATEOAS, niezawodność HTTP]].
5. **System warstwowy** – pośrednicy (proxy, bramy, load balancery, cache) przezroczyści dla klienta.
6. **Kod na żądanie** (opcjonalne) – serwer może przesłać wykonywalny kod (JavaScript).

## Zasób i reprezentacja
- **Zasób** – dowolna rzecz warta nazwania: obiekt biznesowy, kolekcja, wynik obliczeń, relacja, stan procesu.
- **Reprezentacja** – stan zasobu w danym formacie (JSON, XML, HTML, CSV, obraz) w danym momencie. Jeden zasób może mieć wiele reprezentacji.
- Klient **nigdy nie widzi zasobu bezpośrednio**, tylko jego reprezentacje.

## Modelowanie usług REST (metoda Richardsona-Ruby'ego)
1. Określ **zbiór danych** udostępnianych przez usługę.
2. Podziel dane na **zasoby**.
3. Każdemu zasobowi nadaj **URI**.
4. Dla każdego zasobu określ **podzbiór jednolitego interfejsu** (dozwolone metody).
5. Zaprojektuj **reprezentacje** przyjmowane od klienta i wysyłane klientowi.
6. Połącz zasoby **odnośnikami** (hipermedia).
7. Rozważ typowy **przebieg interakcji** (co się dzieje po żądaniu).
8. Rozważ **błędy** (kody statusu, reprezentacje błędów).

### Rodzaje zasobów
- **kolekcja** (`/tasks`) – GET: lista, POST: utworzenie elementu (serwer nadaje URI),
- **element** (`/tasks/17`) – GET, PUT (zastąpienie), PATCH (zmiana części), DELETE,
- **podzasoby** (`/task-lists/3/tasks`) – relacja zawierania,
- **zasoby specjalne / kontrolery** – operacje niedające się sprowadzić do CRUD (zob. zag. 14).

Przykład z własnego projektu: [[Technologie internetowe w przetwarzaniu rozproszonym/TIWPR - Projekt - Dokumentacja usługi]].

## Metody protokołu HTTP
| Metoda | Znaczenie | Bezpieczna | Idempotentna | Ciało żądania | Cache |
|---|---|---|---|---|---|
| **GET** | pobranie reprezentacji | ✔ | ✔ | nie | ✔ |
| **HEAD** | jak GET, bez ciała (metadane, istnienie) | ✔ | ✔ | nie | ✔ |
| **OPTIONS** | dostępne metody (`Allow`), preflight CORS | ✔ | ✔ | opcj. | ✘ |
| **PUT** | utworzenie/zastąpienie zasobu pod **znanym URI** | ✘ | ✔ | tak | ✘ |
| **DELETE** | usunięcie zasobu | ✘ | ✔ | opcj. | ✘ |
| **POST** | utworzenie podrzędnego zasobu (URI nadaje serwer), dopisanie, operacje niestandardowe | ✘ | ✘ | tak | rzadko |
| **PATCH** (RFC 5789) | częściowa modyfikacja (JSON Patch, JSON Merge Patch) | ✘ | ✘ (w ogólności) | tak | ✘ |

- **Bezpieczna** – nie zmienia stanu zasobu (z punktu widzenia klienta); można ją wykonać bez obaw (roboty, prefetch).
- **Idempotentna** – wielokrotne wykonanie daje ten sam **stan serwera** co jednokrotne. Pozwala na bezpieczne **ponawianie** przy zawodnej sieci.
- **PUT vs POST przy tworzeniu**: PUT, gdy klient zna/ustala URI (`PUT /users/jan`); POST do kolekcji, gdy URI nadaje serwer (`POST /users` → `201 Created`, `Location: /users/42`).

## Rola adresów URI
- **Identyfikacja** – każdy zasób ma co najmniej jeden URI; URI jest nazwą zasobu, a nie operacji (rzeczowniki, nie czasowniki: `/orders/5`, nie `/getOrder?id=5`).
- **Adresowalność** – każda interesująca informacja ma własny URI (można ją zapisać w zakładce, przesłać, zbuforować).
- **Hierarchia** wyraża zawieranie: `/users/7/orders/3`.
- **Parametry zapytania** do filtrowania, sortowania i stronicowania: `/tasks?done=false&sort=due&page=2`.
- **Stabilność** – URI nie powinny się zmieniać („Cool URIs don't change”); zmiany sygnalizuje `301 Moved Permanently`.
- **Nieprzezroczystość** dla klienta – klient powinien podążać za odnośnikami, a nie konstruować URI (HATEOAS).
- Konwencje: małe litery, myślniki, liczba mnoga dla kolekcji, brak rozszerzeń (format przez negocjację), wersjonowanie (`/v1/…` lub typ mediów).

## Reprezentacja zasobów
### Negocjacja treści
- **sterowana serwerem (proaktywna)**: klient wysyła `Accept: application/json, application/xml;q=0.8`, `Accept-Language`, `Accept-Encoding`; serwer wybiera i ustawia `Content-Type`, `Vary: Accept`,
- **sterowana klientem (reaktywna)**: serwer zwraca `300 Multiple Choices` z listą wariantów lub odnośniki do różnych URI (`/report.pdf`, `/report.csv`),
- błąd: `406 Not Acceptable`, `415 Unsupported Media Type`.

### Formaty
- **JSON** (`application/json`), **XML**, **HTML** (dla ludzi i hipermediów), formaty hipermedialne: HAL (`application/hal+json`), JSON-LD, Siren, Collection+JSON, Atom (`application/atom+xml`),
- własne typy mediów (`application/vnd.firma.task+json`) – opisują semantykę i mogą wersjonować API.

### Przykład
```http
GET /task-lists/3/tasks/17 HTTP/1.1
Host: api.example.com
Accept: application/json
```
```http
HTTP/1.1 200 OK
Content-Type: application/json
ETag: "a7f3"
Cache-Control: max-age=60

{ "id": 17, "title": "Obrona", "done": false,
  "_links": { "self": { "href": "/task-lists/3/tasks/17" },
              "list": { "href": "/task-lists/3" } } }
```

## Kody statusu
| Klasa | Najważniejsze |
|---|---|
| **2xx** sukces | `200 OK`, `201 Created` (+`Location`), `202 Accepted` (przetwarzanie asynchroniczne), `204 No Content` |
| **3xx** przekierowanie | `301 Moved Permanently`, `303 See Other` (po POST → GET wyniku), `304 Not Modified` (warunkowy GET) |
| **4xx** błąd klienta | `400 Bad Request`, `401 Unauthorized` (brak uwierzytelnienia), `403 Forbidden`, `404 Not Found`, `405 Method Not Allowed` (+`Allow`), `409 Conflict`, `410 Gone`, `412 Precondition Failed`, `415`, `422 Unprocessable Entity`, `429 Too Many Requests` |
| **5xx** błąd serwera | `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable` (+`Retry-After`), `504 Gateway Timeout` |

## Nagłówki istotne dla REST
`Location`, `Content-Type`, `Accept`, `ETag` / `If-None-Match` (warunkowy GET → 304), `If-Match` (optymistyczna kontrola współbieżności → 412), `Last-Modified` / `If-Modified-Since`, `Cache-Control`, `Authorization`, `Link` (odnośniki w nagłówku, RFC 8288).

## REST vs SOAP
| | REST | SOAP / WS-* |
|---|---|---|
| Rodzaj | styl architektury | protokół + stos standardów |
| Interfejs | jednolity (metody HTTP) | dowolne operacje opisane w WSDL |
| Adresowanie | wiele URI (zasoby) | jeden endpoint |
| Formaty | dowolne (JSON, XML…) | XML (koperta SOAP) |
| Transport | HTTP (wykorzystany semantycznie) | HTTP, SMTP, JMS… (HTTP jako tunel) |
| Cache | wbudowany w HTTP | brak (POST) |
| Kontrakt | nieformalny / OpenAPI | formalny WSDL + XSD |
| Bezpieczeństwo, transakcje | TLS, OAuth | WS-Security, WS-AtomicTransaction |

## Zobacz też
- [[14 Architektura zorientowana na zasoby - ROA, HATEOAS, niezawodność HTTP]]
- [[Technologie internetowe w przetwarzaniu rozproszonym/SOA#Implementacja SOA - usługi sieciowe]]
