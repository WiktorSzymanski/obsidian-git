#SRC #Sem1 #PS #TODO

Wszystkie nowe implementacje aplikacji sieciowych powinny działać w trybie dualnym (obsługiwać IPv4 i IPv6)

### Historia
---
Celem IPv6 było rozwiązanie problemu wyczerpania przestrzeni adresowej IPv4. Technologie _CIDR_ oraz _NAT_ są dobrymi rozwiązaniami, choć tylko tymczasowymi. Początek prac nad IPv6 miał miejsce w 1994 roku przez RFC1752. Alternatywną nazwą IPv6 jest IPng (_IP next generation_).

### Budowa
---
Adres IPv6 ma 128 bitów. Podobnie jak dla IPv4, adres IPv6 obejmuje 3 części:
- typ adresu (ang. _prefix_)
- identyfikator sieci (ang. _subnetwork ID_)
- identyfikator interfejsu (ang. _interface ID_)

Długość identyfikatora sieci określa maska w notacji "/". Podobnie jak dla IPv4, adres IPv6 identyfikuje pojedyńczy interfejs podłączony do systemu operacyjnego, a nie cały węzeł. Jeden interfejs może mieć kilka adresów IPv6.


### Zapis adresu
---
Adres IPv6 zapisujemy w postaci szesnastkowej, w ośmiu blokach 2-bajtowych.

###### Przykład
	1235:467A:8BBC ...

Na szczęście adres ten można skracać:
- Przez pominięcie wiodących zer w bloku 2-bajtowym:
	`np. ...`
- Przez zastąpieniem (nejwyżej jeden raz) sekwencji bloków złożonych wyłącznie z zer znakiem "::"
	`np. ...`
### Typy adresów
---
##### Podział ze względu na sposób przesyłania:
- Unicast
		Adres indywidualny. Posiada go tylko jeden interfejs w sieci.
- Multicast
		Adres grupowy. Wiadomość wysyłaną pod ten adres powinny odebrać wszystkie posiadające go interfejsy.
- Anycast
		Podobnie jak powyżej, adres ten nadawany jest wielu interfejsom, ale wiadomość kierowaną pod taki obciążenia, adres powinien odebrać najbliżej położony (według pewnego kryterium) spośród tych inerfejsów. Potencjalnie zastosowania to równoważenie obciążenia i odkrywanie serwerów w sieci.
##### Rodzaje adresów typu unicast:
- global routable - adresy publiczne, zarejestrowane i unikalne
		prefiksy: `2::/3`
		
- site local - adresy prywatne, odpowiednik adresów RFC1918 w IPv4 - obecnie wycofane, choć nadal rozpoznawane w wielu systemach
		prefiks: najczęściej `fec0::/10`
		
- link local - adresy lokalne w obrębie sieci LAN, można się komunkować urządzeniami po sieci lokalnej wykorzystująć link local, a urządzenia podłączone same losują sobie adresy. Prawdopodobieństwo że się powtórzą są na tyle małe że nie ma potrzeby robić tego ręcznie. Stało się to bezużyteczne ponieważ jeśli jeden system operacyjny ma więcej interfejsów to system nie wie którym interfejsem ma wysłać dane. Dlatego do wysyłania pakietów przez link local dodaje się na koniec adresu `%eml` (nazwę interfejsu sieciowego). Oznacza to że chcemy pakiet wysłać na dany adres, prez daną kartę sieciową. Jest to umowne, system operacyjny nie rozpoznaje takiego adresu jako adres poprawny, trzeba sieciowym samemu zaimplementowoać.
		prefiks najczęściej `fe80::/10`

... obrazek

### Tunelowanie IPv6 w IPv4
---
...
### IPv6 w systemie Linux
---
Interfejs typu **sit** (_Simple Internet Transformation_) ...

##### Polecenie [[ip(8)]] obsługuje IPv6 poprzez  argument `-6`.
...

##### Przykładowy tunel w linux
...