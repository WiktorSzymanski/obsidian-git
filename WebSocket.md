#SRC #Sem1 #TIWPR 

### Wiadomości kontrolne
---
#### Ping Pong
### WebSocket Extensions
---
- Wpływają na postać ramki
- #TODO 

### WebSocket Subprotocols
---
- nie zmieniają ramki
- strukturyzują payload
- Klient wysyła:
- `Sec-WebSocket-Protocol: chat`
- Serwer odpowiada:
- #TODO

### Serwery pośredniczące
---
- Detekcja proxy po stronie klienta i wykorzystanie komendy CONNECT protokołu HTTP w celu zestawienia trwałego połączenia z docelowym serwerem
- W przypadku połączeń kodowanych mechanizmem TLS #TODO 

#### WebSocket API
``` js
socket = new WebSocket("<address>");
socket.onopen = function(msg) { ... };
socket.onmessage = function(e) { console.log("Data: " + e.data) };
socket.onclose = function(msg) { ... };
socket.onerror = function(e) { ... };
#TODO
```
### Zastosowania WebSocket
---
- gry
- multimedia
- aplikacje wysoce interaktywne

### WebSocket po stronie serwera
> wymagana jest nowa architektóra serwerów do obsługi dziesiątek tysięcy równoległych połączeń. Stąd stosuje się asynchroniczną (zdarzeniową) obsługę przetwarzania. Np. przy użyciu: node.js, jetty, HttpComponents, Tornado.