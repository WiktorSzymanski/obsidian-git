---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 12
---
# 12. Web Services: SOAP i WSDL – uzupełnienie
---
> Uzupełnienie istniejących notatek [[Technologie internetowe w przetwarzaniu rozproszonym/SOAP]], [[Technologie internetowe w przetwarzaniu rozproszonym/SOAP Messages]], [[Technologie internetowe w przetwarzaniu rozproszonym/WSDL]] o pustą budowę Body/Response, style wiązania i różnice między wersjami.

## Przykład żądania i odpowiedzi SOAP 1.1
```http
POST /StockService HTTP/1.1
Host: example.com
Content-Type: text/xml; charset=utf-8
Content-Length: 312
SOAPAction: "http://example.com/GetPrice"
```
```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
               xmlns:m="http://example.com/stock">
  <soap:Header>
    <m:Transaction soap:mustUnderstand="1">5</m:Transaction>
  </soap:Header>
  <soap:Body>
    <m:GetPrice>
      <m:StockName>IBM</m:StockName>
    </m:GetPrice>
  </soap:Body>
</soap:Envelope>
```
Odpowiedź:
```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
               xmlns:m="http://example.com/stock">
  <soap:Body>
    <m:GetPriceResponse>
      <m:Price>34.5</m:Price>
    </m:GetPriceResponse>
  </soap:Body>
</soap:Envelope>
```

### SOAP Body
- Zawiera **właściwą treść** komunikatu przeznaczoną dla odbiorcy końcowego: wywołanie operacji (element nazwany jak operacja) albo dokument.
- Dzieci elementu `Body` powinny być kwalifikowane przestrzenią nazw aplikacji.
- W odpowiedzi – element wyniku (konwencja: `NazwaOperacjiResponse`) albo `<soap:Fault>`.
- W odróżnieniu od nagłówka Body nie jest przetwarzany przez pośredników (`actor`/`role`).

### SOAP 1.1 vs 1.2
| | SOAP 1.1 | SOAP 1.2 |
|---|---|---|
| Status | W3C Note (2000) | W3C Recommendation (2003, 2007) |
| Przestrzeń nazw | `http://schemas.xmlsoap.org/soap/envelope/` | `http://www.w3.org/2003/05/soap-envelope` |
| Content-Type | `text/xml` + nagłówek `SOAPAction` | `application/soap+xml; action="..."` |
| Pośrednicy | atrybut `actor` | atrybut `role` (+ `relay`) |
| Fault | `faultcode`, `faultstring`, `faultactor`, `detail` | `Code/Value` (+ `Subcode`), `Reason/Text`, `Node`, `Role`, `Detail` |
| Kody błędów | `Client`, `Server` | `Sender`, `Receiver`, `DataEncodingUnknown` |
| HTTP GET | nie | tak (dla operacji bezpiecznych) |
| Model | – | abstrakcyjny model przetwarzania + infoset XML |

## Style wiązania w WSDL (_binding style / use_)
Element `<soap:binding style="rpc|document">` i `<soap:body use="literal|encoded">`:

| Kombinacja | Opis | Status |
|---|---|---|
| **RPC/encoded** | Body zawiera element-operację, parametry z atrybutami typów `xsi:type` (kodowanie SOAP sekcja 5) | przestarzałe, niezgodne z WS-I BP |
| **RPC/literal** | element-operacja, parametry opisane typami XML Schema, bez `xsi:type` | zgodne z WS-I |
| **Document/encoded** | – | nieużywane |
| **Document/literal** | Body zawiera dowolny dokument zgodny ze schematem (części komunikatu jako elementy) | zgodne z WS-I, walidowalne schematem |
| **Document/literal wrapped** | document/literal, gdzie jedyny element Body jest „opakowaniem” nazwanym jak operacja | de facto standard (.NET, JAX-WS) |

- **RPC** – komunikat odwzorowuje wywołanie procedury (nazwa + parametry).
- **Document** – komunikat to dokument biznesowy, a interpretacja należy do usługi (luźniejsze powiązanie).
- **encoded** – serializacja wg reguł SOAP (referencje `href`, tablice), trudna interoperacyjność.
- **literal** – treść dokładnie wg XML Schema.

## Struktura dokumentu WSDL 1.1
```xml
<definitions name="StockQuote" targetNamespace="http://example.com/stock.wsdl"
    xmlns:tns="http://example.com/stock.wsdl" xmlns:xsd1="http://example.com/stock.xsd"
    xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/" xmlns="http://schemas.xmlsoap.org/wsdl/">
  <types>
    <schema targetNamespace="http://example.com/stock.xsd" xmlns="http://www.w3.org/2001/XMLSchema">
      <element name="TradePriceRequest"><complexType><all>
        <element name="tickerSymbol" type="string"/></all></complexType></element>
      <element name="TradePrice"><complexType><all>
        <element name="price" type="float"/></all></complexType></element>
    </schema>
  </types>
  <message name="GetLastTradePriceInput"><part name="body" element="xsd1:TradePriceRequest"/></message>
  <message name="GetLastTradePriceOutput"><part name="body" element="xsd1:TradePrice"/></message>
  <portType name="StockQuotePortType">
    <operation name="GetLastTradePrice">
      <input message="tns:GetLastTradePriceInput"/>
      <output message="tns:GetLastTradePriceOutput"/>
    </operation>
  </portType>
  <binding name="StockQuoteSoapBinding" type="tns:StockQuotePortType">
    <soap:binding style="document" transport="http://schemas.xmlsoap.org/soap/http"/>
    <operation name="GetLastTradePrice">
      <soap:operation soapAction="http://example.com/GetLastTradePrice"/>
      <input><soap:body use="literal"/></input>
      <output><soap:body use="literal"/></output>
    </operation>
  </binding>
  <service name="StockQuoteService">
    <port name="StockQuotePort" binding="tns:StockQuoteSoapBinding">
      <soap:address location="http://example.com/stockquote"/>
    </port>
  </service>
</definitions>
```
Część **abstrakcyjna** (types, message, portType) jest oddzielona od **konkretnej** (binding, service/port).

## WSDL 1.1 vs 2.0
| WSDL 1.1 | WSDL 2.0 |
|---|---|
| `definitions` | `description` |
| `portType` | `interface` (dziedziczenie interfejsów) |
| `port` | `endpoint` |
| `message` + `part` | brak – operacje odwołują się bezpośrednio do elementów schematu |
| 4 typy operacji | **MEP** (_Message Exchange Patterns_): in-only, robust-in-only, in-out, in-optional-out, out-only, out-in… |
| wiązania SOAP 1.1, HTTP, MIME | SOAP 1.2, pełne wiązanie HTTP (także REST) |

## Proces tworzenia usługi
- **contract-first** (top-down): najpierw WSDL + XSD, potem generacja szkieletów (`wsimport`, `wsdl2java`),
- **code-first** (bottom-up): klasa z adnotacjami (`@WebService`, `@WebMethod` w JAX-WS) → WSDL generowany automatycznie.

## Zobacz też
- [[Technologie internetowe w przetwarzaniu rozproszonym/WS-I]], [[Technologie internetowe w przetwarzaniu rozproszonym/Standardy Web Services]]
- [[13 Usługi sieciowe REST]] – porównanie z REST
