---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 15
---
# 15. Asynchroniczna komunikacja w protokole HTTP, protokół WebSocket
---
> HTTP to protokół **żądanie–odpowiedź** inicjowany zawsze przez klienta. Aplikacje interaktywne (czat, gry, notowania, powiadomienia) potrzebują natomiast przesyłania danych **z serwera do klienta w dowolnym momencie**. Stąd techniki asynchroniczne w HTTP, a finalnie osobny protokół **WebSocket**.

## Asynchroniczność po stronie klienta – AJAX
- `XMLHttpRequest` / `fetch()` – żądania wysyłane w tle bez przeładowania strony; odpowiedź obsługiwana w callbacku / `Promise`.
- Nadal inicjuje klient – nie rozwiązuje problemu „push”.

## Techniki „push” nad HTTP (Comet / Reverse Ajax)
### 1. Polling (odpytywanie)
Klient co $T$ sekund wysyła żądanie „czy są nowe dane?”.
- ✔ prosty, działa wszędzie
- ✘ opóźnienie do $T$, dużo pustych żądań (narzut nagłówków, obciążenie serwera)

### 2. Long polling (długie odpytywanie)
Klient wysyła żądanie, a serwer **wstrzymuje odpowiedź**, dopóki nie pojawią się dane (lub timeout). Po odpowiedzi klient natychmiast wysyła następne żądanie.
- ✔ niskie opóźnienie, działa przez proxy/firewalle
- ✘ każde zdarzenie = nowe żądanie HTTP z nagłówkami; serwer trzyma wiele otwartych połączeń (wymaga serwera asynchronicznego – [[16 Asynchroniczna implementacja serwerów usług]]); problemy z uporządkowaniem i utratą komunikatów między żądaniami

### 3. HTTP streaming
Serwer nie kończy odpowiedzi i dopisuje kolejne porcje (`Transfer-Encoding: chunked`), dawniej przez „ukrytą ramkę `iframe`” z kolejnymi `<script>`.
- ✘ buforujące proxy mogą wstrzymywać dane; tylko serwer→klient

### 4. Server-Sent Events (SSE, `EventSource`)
Standard HTML5 dla strumienia zdarzeń **serwer → klient** nad zwykłym HTTP.
```http
GET /events HTTP/1.1
Accept: text/event-stream
Last-Event-ID: 41
```
```
HTTP/1.1 200 OK
Content-Type: text/event-stream
Cache-Control: no-cache

retry: 5000
id: 42
event: price
data: {"symbol":"IBM","price":34.5}

```
```js
const es = new EventSource('/events');
es.addEventListener('price', e => update(JSON.parse(e.data)));
es.onerror = () => { /* przeglądarka sama wznowi połączenie */ };
```
- pola `data:`, `event:`, `id:`, `retry:`; zdarzenia oddzielone pustą linią,
- **automatyczne wznawianie** połączenia z nagłówkiem `Last-Event-ID` (brak utraty zdarzeń),
- ✔ proste, zgodne z HTTP (proxy, HTTP/2 multipleksacja), ✘ tylko jednokierunkowe, tylko tekst (UTF-8), limit połączeń na domenę w HTTP/1.1.

### 5. HTTP/2 Server Push
**Nie** jest mechanizmem powiadomień – pozwala tylko wysłać zasoby zależne w odpowiedzi na żądanie (i jest wycofywany z przeglądarek). Zob. [[Bezpieczeństwo Systemów Rozproszonych/HTTP 2]].

---
## Protokół WebSocket (RFC 6455, 2011)
> **Pełnodupleksowy, trwały kanał komunikacyjny** nad jednym połączeniem TCP, zestawiany przez uzgodnienie HTTP (_upgrade_). Po zestawieniu obie strony mogą w dowolnej chwili wysyłać **komunikaty** (tekstowe lub binarne) z minimalnym narzutem ramki (2–14 bajtów).

Schematy URI: `ws://` (port 80), `wss://` (TLS, port 443). API w przeglądarce: [[Technologie internetowe w przetwarzaniu rozproszonym/WebSocket#WebSocket po stronie serwera|notatka WebSocket]].

### Uzgadnianie (_opening handshake_)
Żądanie klienta:
```http
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Origin: https://example.com
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Extensions: permessage-deflate
```
Odpowiedź serwera:
```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
```
- `Sec-WebSocket-Key` – losowe 16 bajtów w Base64.
- `Sec-WebSocket-Accept` = **Base64(SHA-1(Key + "258EAFA5-E914-47DA-95CA-C5AB0DC85B11"))** – dowód, że serwer rozumie WebSocket; chroni przed pomyłkowym uznaniem zwykłej odpowiedzi HTTP (to nie mechanizm bezpieczeństwa).
- `Origin` – serwer weryfikuje pochodzenie (WebSocket **nie podlega** Same-Origin Policy/CORS – ochrona po stronie serwera, zob. [[28 Polityki Same Origin i Same Site]]).
- **Podprotokół** (`Sec-WebSocket-Protocol`) – semantyka warstwy aplikacji (np. STOMP, MQTT, WAMP); nie zmienia ramek, strukturyzuje payload.
- **Rozszerzenia** (`Sec-WebSocket-Extensions`) – zmieniają ramki, np. `permessage-deflate` (kompresja, bit RSV1).

### Budowa ramki
```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-------+-+-------------+-------------------------------+
|F|R|R|R| opcode|M| Payload len |    Extended payload length    |
|I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
|N|V|V|V|       |S|             |   (if payload len==126/127)   |
| |1|2|3|       |K|             |                               |
+-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
|     Extended payload length continued, if payload len == 127  |
+ - - - - - - - - - - - - - - - +-------------------------------+
|                               | Masking-key, if MASK set to 1 |
+-------------------------------+-------------------------------+
| Masking-key (continued)       |          Payload Data         |
+-------------------------------- - - - - - - - - - - - - - - - +
```
- **FIN** – ostatni fragment komunikatu (komunikat może być podzielony na ramki),
- **RSV1–3** – dla rozszerzeń,
- **opcode**: `0x0` kontynuacja, `0x1` tekst (UTF-8), `0x2` dane binarne, `0x8` **close**, `0x9` **ping**, `0xA` **pong**,
- **MASK** + 4-bajtowy **klucz maskujący** – ramki **klient → serwer muszą być maskowane** (XOR z kluczem), serwer → klient nie. Chroni to przed atakami zatruwania pamięci podręcznej pośredników (_cache poisoning_),
- **długość**: 0–125 wprost; 126 → kolejne 16 bitów; 127 → kolejne 64 bity.

### Ramki kontrolne
- **ping / pong** – utrzymanie połączenia i wykrywanie martwych połączeń (pong musi zawierać dane z pinga); kontrolne ≤ 125 bajtów, niefragmentowane.
- **close** – kod statusu (1000 normalne, 1001 odejście, 1002 błąd protokołu, 1003 nieakceptowany typ, 1006 zerwane bez close, 1008 naruszenie polityki, 1009 za duży komunikat, 1011 błąd serwera) + powód; druga strona odpowiada close, potem zamknięcie TCP.

### Pośrednicy (proxy)
- Klient wykrywa jawne proxy i używa `CONNECT host:port` do zestawienia tunelu.
- Przezroczyste proxy mogą nie rozumieć `Upgrade` → zrywają połączenie; dlatego w praktyce stosuje się `wss://` (TLS ukrywa ruch przed pośrednikami).
- Load balancery muszą wspierać długotrwałe połączenia i `Upgrade`.

### API w przeglądarce
```js
const ws = new WebSocket('wss://example.com/chat', ['chat']);
ws.binaryType = 'arraybuffer';
ws.onopen    = () => ws.send(JSON.stringify({ type: 'hello' }));
ws.onmessage = e => console.log('Dane:', e.data);
ws.onclose   = e => console.log('Zamknięto', e.code, e.reason);
ws.onerror   = e => console.error(e);
ws.bufferedAmount;     // ile czeka w buforze wysyłki
ws.close(1000, 'bye');
```

### Zastosowania
czaty i komunikatory, gry wieloosobowe, notowania giełdowe, wspólna edycja dokumentów, dashboardy na żywo, IoT; biblioteki: Socket.IO (z fallbackiem do long pollingu), STOMP over WebSocket.

## Porównanie technik
| Technika | Kierunek | Opóźnienie | Narzut na komunikat | Zgodność z infrastrukturą HTTP | Uwagi |
|---|---|---|---|---|---|
| Polling | K→S (pull) | do okresu $T$ | pełne żądanie HTTP | pełna | proste, marnotrawne |
| Long polling | S→K | niskie | pełne żądanie/odpowiedź | pełna | wiele otwartych połączeń |
| Streaming | S→K | niskie | mały | problemy z buforującymi proxy | – |
| SSE | S→K | niskie | mały (tekst) | pełna, auto-reconnect | tylko tekst |
| WebSocket | **K↔S** | najniższe | 2–14 B | wymaga `Upgrade` (proxy!) | pełny dupleks, binaria |
| HTTP/2 Push | S→K (zasoby) | – | – | – | nie służy do powiadomień |

## Zobacz też
- [[16 Asynchroniczna implementacja serwerów usług]] – obsługa wielu trwałych połączeń po stronie serwera
- [[QUIC]]
