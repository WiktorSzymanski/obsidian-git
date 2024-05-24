#SRC #Sem1 #TIWPR 

### Web Services Description Language
---
To język oparty na [[XML]] do opisywania serwisów sieciowych i jak uzyskać do nich dostęp. Opisuje operacje i metody jakie dany serwis wystawia. Służy do lokalizowania serwisów sieciowych.

Operacje lub wiadomości są opisane abstrakcyjnie i powiązane z konkretnym protokołem sieciowym i formatem wiadomości aby zdefiniować _endpoint_. Dzięki temu że są one abstrakcyjne, pozwala to na oddzielenie ich od konkretnego wdrożenia sieciowego lub powiązań formatów danych, można użyć tych definicji po raz kolejny.

WSDL jest rozszerzalny, aby umożliwić opis punktu końcowego i jego wiadomości niezależnie od formatów wiadomości lub protokołów sieciowych używanych do komunikacji. Posiada powiązania dla:
- SOAP 1.1
- HTTP GET/POST
- MIME

#### Binding
> wiązanie, to specyfikacja konkretnego protokołu i formatu danych dla pewnego rodzaju portu. Port jest definiowany poprzez powiązanie adresu sieciowego z _binding_-en wielokrotnego użytku.

#### Network Service
>to grupa _endpoint_-ów operujących na wiadomościach zawierających informacje zorientowane na dokument lub procedurę.

### Elementy wykorzystywane przez WSDL
---
- **Types** -  kontener do definiowania typów danych przy użyciu jakiegoś systemu typów (takie jak [[XML]] Schema)
- **Message** - abstrakcyjna, typowana definicja przekazywanych danych
- **Operation** - abstrakcyjny opis akcji wspieranej przez serwis (używający wcześniej zdefiniowanych wiadomości)
- **Port Type** - abstrakcyjny zbiór operacji wspieranych przez jakieś/jakiś punkt końcowy. Tak jak interfejs, udostępnia jakieś operacje.
- **Binding** - specyfikacja konkretnego protokołu i formatu danych dla pewnego portu
- **Port** - pojedynczy _endpoint_ zdefiniowany jako kombinacja _binding_-u i adresu sieciowego
- **Service** - kolekcja powiązanych punktów końcowych (_endpoint_-ów)

### Zasady WSDL
---
- Binding musi określić dokładnie jeden protokół
- Przeciążone/Nadpisane (_overloaded_) metody są identyfikowane przez jawne nazwy parametrów wejścia/wyjścia
- Port nie może określić więcej jak jednego adresu
- Port nie może określać żadnych informacji powiązania (_binding_-u) innych jak informacji o adresie

### Typy operacji
	input - wysyła coś do serwera
	output - odbiera odpowiedź od serwera
---
- **One-way** - _endpoint_ odbiera wiadomość
- **Request-response** - _endpoint_ odbiera wiadomość i wysyła powiązaną odpowiedź
- **Solicit-response** - _endpoint_ wysyła wiadomość i odbiera powiązaną odpowiedź
- **Notification** - _endpoint_ wysyła wiadomość