#SRC #Sem1 #TIWPR

### Budowa wiadomości SOAP
---
- element `<envelope>`
- element `<header>` (opcjonalny)
- element `<body>`
- element `<fault>` (opcjonalny)

### Zasady wiadomości SOAP
---
- Musi być kodowana używając [[XML]]
- Nie może zawierać odniesienia do DTD
- Nie może zawierać XML Processing Instructions

### SOAP Header
---
- Informacje specyficzne dla aplikacji (np. authentication, payment)
- Atrybuty:
	- `actor`
	- `mustUnderstand`
	- `encodingStyle`
### SOAP Body
---

### SOAP Response
---

### Element `<fault>`
---
Jest dzieckiem elementu `<body>`. Może wystąpić tylko raz we wiadomości SOAP.

#### Pod elementy:
- `<faultcode>` - kod identyfikujący błąd
- `<faultstring>` - zrozumiałe dla człowieka wytłumaczenie błędu
- `<faultactor>` - informacja o tym kto spowodował błąd
- `<detail>` - posiada specyficzne dla aplikacji informacje o błędzie dotyczące elementu `<body>`

### SOAP Fault Codes
---
- VersionMismatch - znaleziono błędny `namespace` dla elementu SOAP Envelope
- MustUnderstand - bezpośrednie dziecko elementu `<header>` z atrybutem `mustUnderstand` ustawiony na "1", nie został zrozumiany.
- Client - źle sformułowana wiadomość lub zawierająca błędne informacje
- Server - problem z serwerem więc wiadomość nie została przetworzona

### SOAP HTTP Binding
---
- Żądanie SOAP może być POST-em lub GET-em HTTP
- Żądanie HTTP POST oczekuje co najmniej dwóch nagłówków HTTP
	- `Content-Type`
	- `Content-Lenght`