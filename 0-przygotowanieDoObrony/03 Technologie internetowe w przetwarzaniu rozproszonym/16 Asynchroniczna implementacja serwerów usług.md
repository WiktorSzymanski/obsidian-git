---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 16
---
# 16. Asynchroniczna implementacja serwerów usług
---
> **Asynchroniczna implementacja serwera** oddziela czas trwania połączenia/żądania od czasu zajmowania **wątku**. Wątek nie blokuje się w oczekiwaniu na I/O (bazę danych, inną usługę, zdarzenie do wypchnięcia klientowi), tylko zwalnia się i wraca do obsługi, gdy wynik jest gotowy. Pozwala to obsłużyć **dziesiątki tysięcy równoległych połączeń** (long polling, SSE, WebSocket) małą liczbą wątków.

## Motywacja
### Model synchroniczny: wątek na żądanie (_thread-per-request_)
```
żądanie → wątek z puli → [logika] → [czekanie na BD 200 ms] → [czekanie na usługę 500 ms] → odpowiedź → zwolnienie wątku
```
- Wątek jest **zablokowany** przez cały czas oczekiwania na I/O, choć nie wykonuje pracy.
- Każdy wątek to pamięć stosu (domyślnie ok. 0,5–1 MiB) i koszt przełączania kontekstu.
- Pula wątków (np. 200) ogranicza liczbę jednocześnie obsługiwanych żądań; przy trwałych połączeniach (WebSocket, long polling) każde zajmuje wątek → **wyczerpanie puli**.
- **Problem C10K** (D. Kegel, 1999) – jak obsłużyć 10 000 jednoczesnych połączeń na jednym serwerze.

### Model asynchroniczny (zdarzeniowy)
```
pętla zdarzeń ← (połączenie gotowe) ← epoll/kqueue/IOCP
  ├─ obsłuż fragment żądania, zleć I/O nieblokująco, zarejestruj callback, WRÓĆ do pętli
  └─ (I/O zakończone) → callback: kontynuuj, wyślij odpowiedź
```
- Mała liczba wątków (np. jeden na rdzeń) obsługuje wszystkie połączenia.
- Wymaga **nieblokującego I/O** i **multipleksacji** (select/poll/epoll) – [[60 Obsługa operacji wejścia-wyjścia komunikacji sieciowej]], wzorzec **Reactor** – [[61 Architektury serwerów sieciowych]].
- Zasada: **nigdy nie blokować pętli zdarzeń** (długie obliczenia i blokujące API przenosi się do osobnej puli wątków).

## Abstrakcje programistyczne
| Abstrakcja | Opis | Przykład |
|---|---|---|
| **callback** | funkcja wywołana po zakończeniu operacji | Node.js `fs.readFile(f, (err, data) => …)` – ryzyko „callback hell” |
| **future / promise** | obiekt reprezentujący przyszły wynik; kompozycja `then`, `thenCompose`, `all` | `CompletableFuture`, JS `Promise` |
| **async/await** | składnia sekwencyjna kompilowana do maszyny stanów na promise'ach | C#, JS, Python `asyncio`, Kotlin coroutines |
| **strumienie reaktywne** | asynchroniczne sekwencje z **przeciwciśnieniem** (_backpressure_) | Reactive Streams, Project Reactor (`Mono`, `Flux`), RxJava |
| **aktorzy** | komunikaty do skrzynek, brak współdzielonego stanu | Akka, Erlang – [[45 Modele obliczeń współbieżnych - model aktorów i rachunek pi]] |
| **wątki wirtualne** | lekkie wątki JVM (Project Loom, Java 21) – kod blokujący, pod spodem asynchroniczny | `Executors.newVirtualThreadPerTaskExecutor()` |

## Implementacje po stronie serwera
### Node.js
- jednowątkowa pętla zdarzeń (libuv: epoll/kqueue/IOCP + pula wątków dla operacji plikowych/DNS),
- całe API asynchroniczne; naturalny do WebSocket (biblioteki `ws`, Socket.IO).
```js
http.createServer(async (req, res) => {
  const user = await db.findUser(req.url);   // nie blokuje pętli
  res.end(JSON.stringify(user));
}).listen(8080);
```

### Java Servlet 3.0 – przetwarzanie asynchroniczne
Wątek kontenera zwalnia się, a odpowiedź dokończy inny wątek.
```java
@WebServlet(urlPatterns = "/report", asyncSupported = true)
public class ReportServlet extends HttpServlet {
  protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
    AsyncContext ctx = req.startAsync();          // odłączenie od wątku kontenera
    ctx.setTimeout(30_000);
    executor.submit(() -> {
      String r = generateReport();                // długa operacja w innej puli
      ctx.getResponse().getWriter().write(r);
      ctx.complete();                             // zakończenie odpowiedzi
    });
  }                                               // wątek kontenera wraca do puli
}
```
- **Servlet 3.1** – **nieblokujące I/O**: `ReadListener` (`onDataAvailable`, `onAllDataRead`) i `WriteListener` (`onWritePossible`) – odczyt/zapis strumieni bez blokowania.
- Long polling / Comet: `AsyncContext` przechowywany w kolejce oczekujących klientów, a przy zdarzeniu serwer odpowiada wszystkim.

### JAX-RS 2.0 (REST)
```java
@GET @Path("/orders/{id}")
public void get(@PathParam("id") long id, @Suspended AsyncResponse ar) {
  service.findAsync(id).thenAccept(ar::resume)
                       .exceptionally(t -> { ar.resume(t); return null; });
}
```
Po stronie klienta: `target.request().async().get(InvocationCallback)`; również `SseEventSink` dla SSE.

### Spring WebFlux (reaktywny)
- serwer Netty/Undertow w modelu pętli zdarzeń, typy `Mono<T>` (0..1) i `Flux<T>` (0..N),
- **przeciwciśnienie** – odbiorca sygnalizuje, ile elementów może przyjąć (`request(n)`),
- wymaga reaktywnych sterowników (R2DBC, reactive Mongo); blokujące JDBC niweczy zysk.
```java
@GetMapping(value = "/prices", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
Flux<Price> prices() { return priceService.stream(); }   // SSE
```
- Spring MVC: zwracanie `DeferredResult`, `Callable`, `CompletableFuture`, `SseEmitter`.

### Inne
Netty (event loop groups: boss przyjmuje połączenia, worker obsługuje I/O), Vert.x (reaktor wieloinstancyjny, _verticles_), Jetty (Continuations, a potem Servlet async), Tornado/asyncio/aiohttp (Python), ASP.NET Core (`async Task<IActionResult>`, Kestrel na libuv/sockets), Go (gorutyny + netpoller – kod synchroniczny, wykonanie asynchroniczne).

## Asynchroniczność na poziomie protokołu (usług)
- **202 Accepted + zasób statusu** – długie operacje REST ([[14 Architektura zorientowana na zasoby - ROA, HATEOAS, niezawodność HTTP#Zasoby specjalne]]),
- **callback / webhook** – serwer po zakończeniu wywołuje URL podany przez klienta,
- **WS-Addressing** `ReplyTo` w SOAP – odpowiedź do innego endpointu; wzorce MEP in-only,
- **kolejki komunikatów** (JMS, AMQP) między usługami.

## Porównanie
| Kryterium | Wątek na żądanie | Asynchroniczny / zdarzeniowy |
|---|---|---|
| Liczba połączeń | ograniczona pulą wątków | dziesiątki–setki tysięcy |
| Zużycie pamięci | stos na wątek | stan połączenia (kilka KiB) |
| Przełączanie kontekstu | dużo | mało |
| Model programowania | prosty, sekwencyjny | callbacki/promise/reaktywny – trudniejszy |
| Debugowanie, stos wywołań | czytelne | rozproszone po callbackach |
| Operacje CPU-intensywne | naturalne | blokują pętlę → osobna pula |
| Blokujące biblioteki (JDBC) | bez problemu | niweczą korzyści |
| Zastosowanie | typowe CRUD, krótkie żądania | WebSocket, SSE, long polling, bramy API, proxy, mikroserwisy z dużą liczbą wywołań I/O |

## Zobacz też
- [[15 Asynchroniczna komunikacja HTTP i WebSocket]]
- [[Technologie internetowe w przetwarzaniu rozproszonym/WebSocket#WebSocket po stronie serwera]]
