Dostęp do pdf: będzie później
Zaliczenie:
	Lab - 2 projekty:
		- REST
		- Web sockety
	Wykład - kolokwium zaliczeniowe (pytania otwarte dotyczące ogółu)
	Można przeprowadzić prezentację (45min) zamiast jednego z projektów
	
# HTML
Uźywanie formatowania logicznego jest lepsze ponieważ maszyna lepiej wie jak inerpretować/czytać ich zawartość. Nie narzuca żadnych struktur. 

# XML
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

# CSS & XSL
Może opisywać dokument XML, leczy musi on być bardziej dokładny ze względu na to że XML żadnych własności stylowych nie ma.
XSL - język do wizualizacji XML-a, może zrobić wszystko z prezentacją pliku XML dzięki możliwości transformacji. Może doknać transformacji tabeli w wykres.
XSL-FO - domyślnie inerpretuje plik XML, nie ma przeglądarki która obsługuje te technologię.

# SVG
Jest aplikacją XML, ponieważ jest to zapis tekstowy, maszyna jest wstanie przeczytać tekst znajdujący się w pliku SVG. Łatwo skalowalny pod dowolnej wielkości ekran.

# MathML
Szybko staje się nieczytelny. Bez narzędzia nie ma podejścia do wzorów.

# DocBook
Standard zapisu dokumentacji technicznej. Są różne narzędzia do konwersji do odpowiednego medium (HTML, PDF czy man).

# Przetwarzanie dokumentów XML
- Document Object Model - wczytuje cały plik do pamięci, nie jest to dobre gdy mamy duży plik
- SAX - interfejs strumieniowy, przepływa przez plik
- StAX - też strumieniowy ale sterowanie jest po naszej stonie nie inerpretera

# XHTML
podział na 28 modułów aby usprawnić rozwój poszczególnych funkcjonalności. Modularyzacja pozwala łączyć XHTML z np. MathML. Zamknięto projekt XHTML 2.0 bo nie było chętnych do implementowania tego standardu.

# HTML 5 
### Multimedia
Element </video/> wyparł flash 
Element </audio/> do reprezentacji zapisu dźwięku.

Problem z formatami danych, z powodu braku standardu.