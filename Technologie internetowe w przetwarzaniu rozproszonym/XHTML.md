---
up: 
tags:
  - TechnologieInternetoweWPrzetwarzaniuRozproszonym
---

- XHTML 1.0 - W3C Recomendation (styczeń 2000)
- XHTML używa tych samych znaczników co [[HTML]] 4.01
- Pełna zgodność z [[XML]]
- Rozróżnianie wielkich i małych liter
- Poprawność formatowania
##### Przykład
```
<!DOCTYPE html PUBLIC "~//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml/DTD/xhtml1-strict.dtd"

<html xmlns="http://www.w3.org/1999/xhtml">
	<head>
		<title>Przykład dokumentu XHTML</title>
	</head>
	<body>
		<p>Akapit tekstowy.</p>
	</body> 

</html>
```

### HTML/XHTML - jaki typ MIME
---
- Deklaracja typu dokumentu (DOCTYPE) jest niewystarczająca

`text/html` - tryb HTML, tryb zgodności wstecznej, akceptacja błędnego formatowania
`application/xhtml+xml` - tryb XML, wymaga poprawne formatowanie, możliwość stosowania przestrzeni nazwi

- 99% stron XHTML serwowanych jest jako `text/html`

### Modularyzacja XHTML
---
Wprowadzono podział na 28 modułów aby usprawnić rozwój poszczególnych funkcjonalności. Modularyzacja pozwala łączyć XHTML z np. MathML. 

| Moduł     | Opis                                              |
| --------- | ------------------------------------------------- |
| Hypertext | Element `<a>`                                     |
| List      | Elementy `<ol>`, `<li>`, `<dd>`, `<dt>`, `<dl>`   |
| Structure | Elementy  `<html>`, `<head>`, `<title>`, `<body>` |
| Tables    | Elementy do obsługi tabel                         |
| Text      | Elementy blokowe, m.in.: `<p>`, `<h1>`            |
| ...       | ...                                               |
XHTML Basic - minimalny XHTML dla potrzeb np. tel. kom., PDA

### XHTML 1.1
---
Wersja 1.1 to przeformułowany XHTML 1.0 strict z użyciem modułów. Dodano XFrames - standard uzupełniający XHTML o ulepszoną obsługę ramek.

Zamknięto projekt [[XHTML 2.0]] bo nie było chętnych do implementowania tego standardu.
