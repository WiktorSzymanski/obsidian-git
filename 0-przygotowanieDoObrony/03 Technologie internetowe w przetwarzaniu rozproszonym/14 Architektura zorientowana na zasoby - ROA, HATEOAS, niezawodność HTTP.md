---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 14
---
# 14. Architektura zorientowana na zasoby (ROA), RPC a zasoby, zasoby specjalne, HATEOAS, niezawodność HTTP
---
> **ROA** (_Resource-Oriented Architecture_, L. Richardson, S. Ruby, „RESTful Web Services”, 2007) to konkretna, praktyczna architektura realizująca styl REST w oparciu o HTTP i URI. Opisuje, jak projektować usługi, w których **wszystko jest zasobem**.

Podstawy REST: [[13 Usługi sieciowe REST]].

## Pojęcia ROA
- **zasób**, **URI** (nazwa i adres zasobu), **reprezentacja**, **odnośnik** między zasobami.

## Cztery własności ROA
1. **Adresowalność** (_addressability_) – każdy zasób ma własny URI. Dzięki temu można go zapamiętać, przesłać, zbuforować, a klienci mogą łączyć zasoby.
2. **Bezstanowość** (_statelessness_) – każde żądanie jest niezależne. Rozróżnia się:
   - **stan aplikacji** (gdzie klient jest w przebiegu interakcji) – przechowywany **u klienta**,
   - **stan zasobu** – przechowywany **na serwerze**, taki sam dla wszystkich klientów.

   Skutki: skalowalność (dowolny serwer obsłuży żądanie), łatwiejsze buforowanie i odtwarzanie po awarii.
3. **Połączalność** (_connectedness_) – reprezentacje zawierają **odnośniki** do innych zasobów i opisują dozwolone przejścia. Klient porusza się po usłudze jak po stronach WWW.
4. **Jednolity interfejs** (_uniform interface_) – ten sam, ograniczony zbiór metod HTTP dla wszystkich zasobów o dobrze zdefiniowanej semantyce (bezpieczeństwo, idempotencja).

## Styl RPC a zasoby
### Usługi w stylu RPC / hybrydowe
- **RPC** (np. SOAP, XML-RPC): jeden endpoint, nazwa operacji w treści (`<getOrder>`) – metoda HTTP (zwykle POST) i URI nie niosą informacji.
- **Hybryda REST-RPC** („REST-like”): operacja zakodowana w URI (`GET /api?method=deleteUser&id=5`), często GET z efektami ubocznymi – łamie semantykę HTTP (niebezpieczne dla cache, robotów, prefetch).
- W stylu RPC **informacja o zakresie** (który obiekt) i **informacja o metodzie** (co zrobić) ukryte są w treści; w ROA zakres niesie **URI**, a metodę – **metoda HTTP**.

### Model dojrzałości Richardsona
| Poziom | Opis | Przykład |
|---|---|---|
| **0** – „bagno POX” | HTTP jako tunel, jeden URI, jedna metoda | SOAP, XML-RPC |
| **1** – zasoby | wiele URI (zasoby), ale nadal jedna metoda | `POST /orders/5` z akcją w treści |
| **2** – czasowniki HTTP | poprawne użycie metod i kodów statusu | `GET/PUT/DELETE /orders/5`, `201 Created` |
| **3** – kontrolki hipermedialne | HATEOAS – odnośniki opisują możliwe akcje | reprezentacje z `_links` |

### Przekształcanie operacji RPC w zasoby
Operacje w stylu czasownika zamienia się na **rzeczowniki** (zasoby), którymi manipuluje się jednolitym interfejsem:
| Styl RPC | ROA |
|---|---|
| `transferMoney(a, b, 100)` | `POST /transfers` z treścią `{from, to, amount}` → `201 /transfers/88` |
| `approveOrder(5)` | `PUT /orders/5/approval` lub `PATCH /orders/5 {"status":"approved"}` |
| `searchBooks("rest")` | `GET /books?q=rest` (wyniki wyszukiwania jako zasób) |
| `login(user, pass)` | `POST /auth/tokens` (token jako zasób) |
| `mergeLists(1, 2)` | `POST /task-lists-merges` |

## Zasoby specjalne
Zasoby, które nie odpowiadają bezpośrednio encjom dziedziny, ale reifikują **operacje, relacje lub procesy**:
- **Zasoby-kontrolery (algorytmiczne)** – wynik obliczenia lub akcja: `/search?q=`, `/routes?from=A&to=B`, `/primes/1000`.
- **Transakcje jako zasoby** – `POST /transactions` tworzy transakcję, kolejne żądania `PUT /transactions/11/accounts/checking` modyfikują jej zasoby, a `PUT /transactions/11 {"committed":true}` zatwierdza (lub `DELETE` wycofuje). Pozwala na atomowość bez utrzymywania sesji.
- **Operacje wsadowe** (_batch_) – `POST /jobs` z listą operacji; zasób zadania z postępem.
- **Scalenia i relacje** – `POST /task-lists-merges`, `/friendships/7`.
- **Zasoby jednorazowe / tokeny** – np. token zmiany statusu (`/tasks/{id}/toggles/{token}`) albo zasób z **POST Once Exactly**; ich konsumpcja jest jednorazowa, co pozwala uczynić operację idempotentną.
- **Zasoby asynchroniczne** – długie operacje: `POST /reports` → `202 Accepted`, `Location: /reports/queue/5`; klient odpytuje `GET /reports/queue/5` (status), po zakończeniu `303 See Other` → `/reports/5`.
- **Zasoby-widoki / agregaty** – `/dashboard`, `/users/7/summary`.
- **Zasób główny / punkt wejścia** (_home document_) – `GET /` z odnośnikami do wszystkich kolekcji (start HATEOAS).

Przykłady z własnego projektu: `/api/auth/tokens`, `/api/tasks-lists-merges`, `/api/…/toggles/{token}` w [[Technologie internetowe w przetwarzaniu rozproszonym/TIWPR - Projekt - Dokumentacja usługi]].

## HATEOAS
> **Hypermedia As The Engine Of Application State** – klient przechodzi między stanami aplikacji wyłącznie przez **odnośniki i kontrolki zawarte w reprezentacjach** otrzymanych od serwera. Nie korzysta z wiedzy o strukturze URI zapisanej w kodzie.

- Klient zna tylko **punkt wejścia** i **typy mediów / relacje odnośników** (`rel`).
- Serwer wskazuje **aktualnie dozwolone akcje**: odnośnik `cancel` pojawia się tylko dla zamówienia, które można anulować.
- Korzyści: luźne powiązanie (serwer może zmieniać URI), samodokumentowanie, ewolucja API bez łamania klientów, logika przejść stanów po stronie serwera.
- Formaty: HAL, JSON-LD/Hydra, Siren (akcje z metodą i polami), Atom, HTML (formularze = kontrolki), nagłówek `Link`.

```json
{
  "id": 5, "status": "unpaid", "total": 120.0,
  "_links": {
    "self":    { "href": "/orders/5" },
    "payment": { "href": "/orders/5/payment" },
    "cancel":  { "href": "/orders/5", "method": "DELETE" },
    "items":   { "href": "/orders/5/items" }
  }
}
```
Po opłaceniu reprezentacja nie zawiera już `payment` ani `cancel`, ale np. `receipt`.

## Niezawodność HTTP
HTTP działa nad zawodną siecią: klient, który nie dostał odpowiedzi, **nie wie**, czy żądanie dotarło i zostało wykonane.

### Idempotencja jako podstawa
- **GET, HEAD, PUT, DELETE** są idempotentne → po timeoucie można je **bezpiecznie powtórzyć** (semantyka co-najmniej-raz daje efekt dokładnie-raz).
- **POST nie jest idempotentny** → ponowienie może np. utworzyć dwa zamówienia lub dwukrotnie obciążyć konto.

### Techniki uczynienia operacji niezawodnymi
1. **PUT zamiast POST** – klient sam generuje identyfikator (np. UUID) i tworzy zasób `PUT /orders/7c9e…`; powtórzenie nadpisuje ten sam zasób.
2. **Wzorzec POST → PUT (dwufazowe tworzenie)** – `POST /orders` tworzy pusty zasób i zwraca URI (powtórzenie tworzy co najwyżej porzucony, pusty zasób, który można usunąć); następnie idempotentny `PUT` na ten URI zapisuje treść.
3. **POST Once Exactly (POE)** (M. Nottingham, draft IETF 2005) – serwer oznacza zasób nagłówkiem `POE: 1`, zwracanym przy `GET`/`OPTIONS`, i udostępnia **jednorazowy URI**. Pierwszy `POST` na ten URI wykonuje operację, a każdy kolejny zwraca `405 Method Not Allowed` (z informacją, że już wykonano). Klient może więc ponawiać POST aż do otrzymania odpowiedzi, a operacja wykona się co najwyżej raz. Por. [[Technologie internetowe w przetwarzaniu rozproszonym/POST once exactly]].
4. **Klucz idempotencji** (współcześnie, np. Stripe, draft IETF `Idempotency-Key`) – klient dołącza unikalny nagłówek; serwer zapamiętuje wynik dla klucza i przy powtórzeniu zwraca zapisaną odpowiedź.
5. **Żądania warunkowe – kontrola współbieżności** (problem **utraconej aktualizacji**, _lost update_):
   ```http
   GET /docs/1           →  200 OK, ETag: "v7"
   PUT /docs/1  If-Match: "v7"   →  204 (zapisano, nowe ETag "v8")
   PUT /docs/1  If-Match: "v7"   →  412 Precondition Failed (ktoś zmienił w międzyczasie)
   ```
   `If-None-Match: *` przy PUT – utwórz tylko jeśli nie istnieje (unikanie nadpisania).
6. **Zasoby asynchroniczne** (`202 Accepted` + zasób statusu) – długie operacje nie przekraczają timeoutów; klient odpytuje stan.
7. **Retry z wykładniczym odczekiwaniem** dla `503` + `Retry-After`, `429`.
8. **Transakcje jako zasoby** – dla wielu zasobów wymagających atomowości (wyżej).

### HTTP a warstwa transportowa
- HTTP/1.1 keep-alive i **pipelining** (tylko dla metod idempotentnych – przy zerwaniu połączenia żądania są ponawiane), HTTP/2 multipleksacja – [[Bezpieczeństwo Systemów Rozproszonych/HTTP 2]].
- Przeglądarka i proxy mogą automatycznie ponowić żądania idempotentne, ale nie POST.

## Zobacz też
- [[15 Asynchroniczna komunikacja HTTP i WebSocket]]
- [[08 Podejścia do budowy systemów rozproszonych#3. Zdalne wywołanie procedur (RPC)]] – semantyki wywołań
