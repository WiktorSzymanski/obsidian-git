---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 28
---
# 28. Polityki Same Origin oraz Same Site
---
> **Same-Origin Policy (SOP)** to podstawowy mechanizm bezpieczeństwa przeglądarki. Skrypt załadowany z jednego **pochodzenia** nie może **odczytywać** danych należących do innego pochodzenia (odpowiedzi HTTP, DOM innego okna, magazynu). Chroni to dane użytkownika na stronie A (np. banku) przed złośliwym skryptem ze strony B, choć obie są otwarte w tej samej przeglądarce z tymi samymi ciasteczkami.

**Same-Site** to szersze pojęcie (**witryna** zamiast pochodzenia). Używa go m.in. atrybut ciasteczek **SameSite**, który ogranicza wysyłanie ciasteczek w żądaniach międzywitrynowych (ochrona przed CSRF).

## Pochodzenie (_origin_)
**Pochodzenie = schemat + host + port** (RFC 6454).

Porównanie z `https://www.bank.pl/konto`:
| URL | To samo pochodzenie? | Dlaczego |
|---|---|---|
| `https://www.bank.pl/przelewy?x=1` | ✔ | inna ścieżka/zapytanie nie ma znaczenia |
| `http://www.bank.pl/konto` | ✘ | inny schemat |
| `https://www.bank.pl:8443/` | ✘ | inny port (domyślnie 443) |
| `https://api.bank.pl/` | ✘ | inny host (subdomena) |
| `https://bank.pl/` | ✘ | inny host |

Pochodzenie **nieprzezroczyste** (`null`) – dokumenty z `data:`, `file:`, sandboxowane iframe.

## Co ogranicza SOP
Ogólna zasada: **zapis/osadzanie międzypochodzeniowe dozwolone, odczyt zabroniony**.
| Działanie | Między pochodzeniami |
|---|---|
| **Osadzanie** zasobów: `<img>`, `<script src>`, `<link rel=stylesheet>`, `<video>`, `<iframe>`, czcionki (z CORS) | ✔ dozwolone (wynik nie jest czytelny dla skryptu: nie odczyta pikseli obrazu z canvas, treści iframe) |
| **Nawigacja** / wysłanie formularza (`<form action=https://inna>`), przekierowania | ✔ dozwolone (**zapis**) → dlatego istnieje **CSRF** |
| Wysłanie żądania `fetch`/XHR | ✔ żądanie **wychodzi** (dla prostych żądań), ale **odczyt odpowiedzi** zablokowany bez CORS |
| Odczyt DOM innego okna/iframe (`iframe.contentWindow.document`) | ✘ |
| Dostęp do `localStorage`, `sessionStorage`, IndexedDB, Cache API | ✘ (izolowane per pochodzenie) |
| Ciasteczka | **inne reguły** – zakres domena + ścieżka, bez portu (i historycznie bez schematu) → ciasteczka są **wysyłane** z żądaniami międzywitrynowymi (stąd SameSite) |
| WebSocket | ✘ **nie podlega SOP** – przeglądarka wysyła nagłówek `Origin`, serwer musi go sprawdzić ([[15 Asynchroniczna komunikacja HTTP i WebSocket]]) |

## Kontrolowane łagodzenie SOP
### CORS (_Cross-Origin Resource Sharing_, Fetch Standard)
Serwer **jawnie zezwala** wybranym pochodzeniom na odczyt odpowiedzi.

**Żądanie proste** (_simple request_): metoda GET/HEAD/POST, tylko nagłówki z białej listy, `Content-Type` ∈ {`application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain`}:
```http
GET /api/dane HTTP/1.1
Origin: https://app.example.com
```
```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://app.example.com
Vary: Origin
```
Przeglądarka udostępnia odpowiedź skryptowi tylko, jeśli `Access-Control-Allow-Origin` pasuje (albo `*`).

**Żądanie wstępne** (_preflight_) dla pozostałych (PUT/DELETE, `Content-Type: application/json`, własne nagłówki np. `Authorization`):
```http
OPTIONS /api/dane HTTP/1.1
Origin: https://app.example.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Content-Type, Authorization
```
```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 600
```
Dopiero po pozytywnej odpowiedzi przeglądarka wysyła właściwe żądanie.

**Poświadczenia** (ciasteczka, uwierzytelnianie HTTP): klient `fetch(url, {credentials: 'include'})`, serwer musi zwrócić `Access-Control-Allow-Credentials: true` i **konkretne** pochodzenie (**nie** `*`).

**Uwagi bezpieczeństwa**: CORS **nie jest ochroną serwera** – nie blokuje żądań z curl ani wysłania żądania (zapis). Błędy konfiguracji: odbijanie dowolnego `Origin` z `Allow-Credentials: true`, zaufanie do `null`, słabe dopasowanie regex (`bank.pl.attacker.com`). Nagłówek `Access-Control-Expose-Headers` wskazuje, które nagłówki odpowiedzi skrypt może odczytać.

### `postMessage` – komunikacja między oknami
```js
// nadawca
iframe.contentWindow.postMessage({ typ: 'token', v: 1 }, 'https://widget.example.com');
// odbiorca
window.addEventListener('message', e => {
  if (e.origin !== 'https://app.example.com') return;   // obowiązkowa weryfikacja!
  handle(e.data);
});
```
Zawsze podawać konkretne `targetOrigin` (nie `*`) i sprawdzać `event.origin`.

### Techniki przestarzałe
- **JSONP** – `<script src="https://api/x?callback=f">` (osadzanie skryptu omija SOP); ryzykowne (wykonanie dowolnego kodu dostawcy, tylko GET, wycieki danych),
- **`document.domain`** – obniżenie domeny do wspólnego rodzica (`a.example.com` i `b.example.com` → `example.com`); wycofywane,
- proxy po stronie serwera (żądanie z własnego pochodzenia).

### Dodatkowe mechanizmy izolacji
- **CORP** (`Cross-Origin-Resource-Policy: same-origin | same-site | cross-origin`) – zasób nie może być osadzony przez inne pochodzenia/witryny,
- **COOP** (`Cross-Origin-Opener-Policy: same-origin`) – izolacja kontekstu przeglądania od okien otwartych z innych pochodzeń (ochrona przed XS-Leaks, Spectre),
- **COEP** (`Cross-Origin-Embedder-Policy: require-corp`) – wymaganie jawnej zgody osadzanych zasobów; COOP+COEP = _cross-origin isolated_ (dostęp do `SharedArrayBuffer`),
- **CSP** (`Content-Security-Policy`) – skąd wolno ładować skrypty, `frame-ancestors` (kto może osadzić stronę – ochrona przed clickjackingiem, następca `X-Frame-Options`),
- **Fetch Metadata** – nagłówki `Sec-Fetch-Site: same-origin | same-site | cross-site | none`, `Sec-Fetch-Mode`, `Sec-Fetch-Dest` pozwalają serwerowi odrzucać nieoczekiwane żądania międzywitrynowe.

---
## Witryna (_site_) i Same-Site
**Witryna = schemat + zarejestrowana domena (eTLD+1)** („schemeful same-site”).
- **eTLD** (_effective top-level domain_) – sufiks publiczny z **Public Suffix List**: `com`, `pl`, `com.pl`, `github.io`, `co.uk`.
- **eTLD+1** – eTLD + jedna etykieta: `bank.pl`, `example.com.pl`, `uzytkownik.github.io`.

| Porównanie z `https://www.bank.pl` | same-origin | same-site |
|---|---|---|
| `https://api.bank.pl` | ✘ | ✔ (ta sama eTLD+1 `bank.pl`) |
| `https://www.bank.pl:8443` | ✘ | ✔ (port nie ma znaczenia dla witryny) |
| `http://www.bank.pl` | ✘ | ✘ (schemat – schemeful same-site) |
| `https://bank.com.pl` | ✘ | ✘ |
| `https://alice.github.io` vs `https://bob.github.io` | ✘ | ✘ (`github.io` to sufiks publiczny) |

Same-site jest **słabsze** niż same-origin: subdomeny tej samej organizacji są „tą samą witryną”, więc przejęta subdomena (np. zapomniany CNAME) może atakować główną stronę.

## Atrybut ciasteczek SameSite (RFC 6265bis)
```http
Set-Cookie: sid=abc123; Secure; HttpOnly; SameSite=Lax; Path=/
```
| Wartość | Wysyłane z żądaniami międzywitrynowymi | Zastosowanie |
|---|---|---|
| **Strict** | **nigdy** – także przy kliknięciu linku z innej witryny (użytkownik trafia „niezalogowany”) | ciasteczka operacji wrażliwych (np. token przelewu) |
| **Lax** | **tylko** przy nawigacji najwyższego poziomu metodą **bezpieczną** (GET kliknięciem linku, przekierowanie); **nie** przy POST formularza, `<img>`, `<iframe>`, `fetch` | domyślna, rozsądna ochrona sesji |
| **None** | zawsze; **wymaga `Secure`** | osadzane widżety, SSO w iframe, reklamy/śledzenie |

- **Domyślne zachowanie** nowoczesnych przeglądarek (Chrome od 2020): brak atrybutu → traktowane jak **Lax** (z tymczasowym wyjątkiem „Lax+POST” – POST dozwolony przez 2 minuty po ustawieniu ciasteczka, dla przepływów SSO).
- **Związek z CSRF**: atak CSRF polega na tym, że złośliwa witryna powoduje wysłanie żądania do banku **z ciasteczkiem sesji ofiary**. SameSite=Lax/Strict sprawia, że międzywitrynowy POST **nie niesie ciasteczka** → żądanie nieuwierzytelnione. Szczegóły: [[29 Podatności webowe - XSS, SQLi, CSRF#CSRF – Cross-Site Request Forgery]].
- **Ograniczenia**: nie chroni przed atakami z **tej samej witryny** (przejęta subdomena, XSS), przed GET zmieniającym stan przy Lax, w starych przeglądarkach → stosować razem z tokenami anty-CSRF.
- Pokrewne: prefiksy `__Host-` (wymaga `Secure`, `Path=/`, bez `Domain` – ciasteczko przypięte do hosta) i `__Secure-`; **ciasteczka partycjonowane** (CHIPS, `Partitioned`) – ciasteczka stron trzecich osobne dla każdej witryny najwyższego poziomu; wycofywanie ciasteczek stron trzecich.

## SOP vs SameSite – podsumowanie
| | Same-Origin Policy | SameSite (ciasteczka) |
|---|---|---|
| Granica | pochodzenie (schemat+host+port) | witryna (schemat+eTLD+1) |
| Chroni przed | **odczytem** danych innego pochodzenia przez skrypt | **wysyłaniem** poświadczeń w żądaniach międzywitrynowych (CSRF, XS-Leaks) |
| Egzekwuje | przeglądarka (domyślnie zawsze) | przeglądarka wg atrybutu ustawionego przez serwer |
| Łagodzenie | CORS, postMessage | `SameSite=None; Secure` |
| Nie chroni przed | CSRF (zapis dozwolony), XSS (skrypt działa w tym samym pochodzeniu) | XSS, atakami z tej samej witryny |
