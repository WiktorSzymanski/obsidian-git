#SRC #Sem1 #AR #TODO 

##### Zastosowania
---
- Rozproszona pamięć współdzielona
- Systemy plików (AFS, NFS)


[[Model Spójności]]


### Aktualizacja replik
---
###### Transfer stanu (ang. _state transfer_)
	serwery przekazują między sobą kompletny stan obiektu po modyfikacji
- małe opóźnienia, nieefektywność w przypadku małych zmian
- optymalizacja: przesyłanie zmian stanu obiektów (_diffs_)
- opłacalne przy znacznych modyfikacjach lub przy agregacji wielu modyfikacji

###### Transfer przetwarzania (ang. _operation transfer_)
	serwery powielają przetwarzanie wykonane na innych serwerach (wymagany determinizm przetwarzania). 
- zapotrzebowanie na dodatkową moc obliczeniową
- opłacalne przy małych modyfikacjach dużych obiektów

Transfer stanu jest prostszy do implementacji ze względu na przesłanie stanu i nadpisanie wartości. Transfer przetwarzania może być problematyczny przy operacjach procentowej zmiany wartości. Kolejność przetwarzania ma wtedy znaczenie, co komplikuję poprawną replikację.

###### System scentralizowany
	odczyt zawsze zwraca wartość 

...

### Koordynacja pracy serwerów
---
###### Rozwiązanie z tokenem
- pomagają przy uspójnieniu kolejności wykonywania operacji na równoległych serwerach.