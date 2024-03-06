#SRC #Sem1 #TIWPR #wykład 

Uproszczona o zbyt specyficzne właściwości SGML. Nie rozróżnia znaczników, jedyne co jest ważne to zagnieżdzenia. Niezbędna jest definicja wyświetlania. Widziany w ten sam sposób przez człowieka i maszynę. Stosowany nie tylko do aplikacji webowych ale także do wymiany/przekazywania danych. Wymusza UNICODE. Aplikacja XML - język opisywany przez XML. Wymusza pewną strukturę, trzeba zamykać znaczniki. 
Document Type Definition - określa co w jakim miejscu może być. 
XML Schema - pozwala na definiowanie jaki tekst co oznacza, np. coś jest liczbą zmiennoprzecinkową.
Definicja dokumentów **fully-normalized** sprowadza dokument do postaci kanonicznej co pozwala porównać kod binarny dwóch XML-ów by sprawdzić czy są takie same (gdyby gdzieś były dodatowe znaki białe bez tego nie było by to oczywiste).

Binarna reprezentacja dokumentów XML pozwala zmiejszyć wielkość dokumentu XML - zamiast znacznika <data_urodzenia>.

Technologie do reprezentacji binarnej:
- Fast Infoset
- EXI - Efficient XML Interchange
- VTD-XML : XML + index
Minus że mamy teraz postać binarną zamiast tekstowej.

Dokument XML może być zwizualizowany przez CSS lub [[XSL]]. 
Przy opisywaniu styli CSS-em, opis ten musi być bardziej dokładny (niż w przypadku [[HTML]]) ze względu na to że XML nie ma żadnych domyślnych własności stylowych.

#### SVG
Jest aplikacją XML, ponieważ jest to zapis tekstowy, maszyna jest wstanie przeczytać tekst znajdujący się w pliku SVG. Łatwo skalowalny pod dowolnej wielkości ekran.

#### MathML
Szybko staje się nieczytelny. Bez narzędzia nie ma podejścia do wzorów.

#### DocBook
Standard zapisu dokumentacji technicznej. Są różne narzędzia do konwersji do odpowiednego medium (HTML, PDF czy man).

#### Przetwarzanie dokumentów XML
- Document Object Model - wczytuje cały plik do pamięci, nie jest to dobre gdy mamy duży plik
- SAX - interfejs strumieniowy, przepływa przez plik
- StAX - też strumieniowy ale sterowanie jest po naszej stonie nie inerpretera