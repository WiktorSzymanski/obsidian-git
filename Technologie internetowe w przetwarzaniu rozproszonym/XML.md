---
up: 
tags:
  - TechnologieInternetoweWPrzetwarzaniuRozproszonym
---

### Extensible Markup Language
---
- Metajęzyk -> język opisu języków
- Uproszczony o zbyt specyficzne właściwości SGML (_Standard Generalized Markup Language_).
- Nie rozróżnia znaczników, jedyne co jest ważne to zagnieżdzenia.
- Niezbędna jest definicja wyświetlania.
- Widziany w ten sam sposób przez człowieka i maszynę.
- Stosowany nie tylko do aplikacji webowych ale także do wymiany/przekazywania danych.
- Wymusza UNICODE.
- Aplikacja XML -> język opisywany przez XML.
- Wymusza pewną strukturę, trzeba zamykać znaczniki.


Wykorzystywany do:
- [[SVG]]
- [[MathML]]
- [[DocBook]]
- WML
- OASIS OpenDocument Format
- Microsoft Open Office XML

### Poprawność dokumentu (_ang. valid_)
---
##### Document Type Definition
- określa co w jakim miejscu może być. 

##### XML Schema
- pozwala na definiowanie jaki tekst co oznacza, np. coś jest liczbą zmiennoprzecinkową.

##### RELAX NG (REgular LAnguage for XML Next Generation)

### Wersje
---
###### XML 1.0
- specyfikacja z roku 1998
- oparty na Unicode 2.0
- dopuszczalne są tylko zdefiniowane znaczniki

###### XML 1.1
- third edition (II/2004)
- oparty na Unicode 4.x
- dopuszczalne są wszystkie znaki specjalne 
- dopuszczalne jest również użycie bezpośrednio znaków sterujących od `0x01` do `0x1F`

Definicja dokumentów **fully-normalized** sprowadza dokument do postaci kanonicznej co pozwala porównać kod binarny dwóch XML-ów by sprawdzić czy są takie same (gdyby gdzieś były dodatowe znaki białe bez tego nie było by to oczywiste).


### XML Information Set
---
Abstrakrycjna definicja dokumentów XML -> bez odniesień do tekstowej reprezentacji

Pojęcia:
- information set (tree)
- information item (node)

Binarna reprezentacja dokumentów XML pozwala zmiejszyć wielkość dokumentu, swobodny dostęp, łatwiejsze i szybsze przetwarzanie, oraz indeksowanie.

Technologie do reprezentacji binarnej:
- Fast Infoset
- EXI - Efficient XML Interchange
- VTD-XML : XML + index

Minus że mamy teraz postać binarną zamiast tekstowej -> nie możemy etytować ręcznie.

### Prezentacja dokumentów XML
---
Dokument XML może być zwizualizowany przez CSS lub [[XSL]].
Przy opisywaniu styli CSS-em, opis ten musi być bardziej dokładny (niż w przypadku [[HTML]]) ze względu na to że XML nie ma żadnych domyślnych własności stylowych. CSS posiada własność `display`.

### Przetwarzanie dokumentów XML
---
- Document Object Model - wczytuje cały plik do pamięci, nie jest to dobre gdy mamy duży plik
- SAX - interfejs strumieniowy, przepływa przez plik
- StAX - też strumieniowy ale sterowanie jest po naszej stonie nie inerpretera
- XQuerry
- XPath