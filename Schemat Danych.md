---
up: "[[LDAP]]"
class: ZSK
---
# Schemat danych
---

>opisuje jakiego typu informacje z jakiego typu atrybutami mogą być przechowywane przez nasz serwer usługi katalogowej. Ustalenie schematu danych ułatwia zarządzanie informacją, wymusza jej uspójnienie, porządkuje te informacje i redukuje ich nadmiarowość. Ułatwia tworzenie aplikacji, poprzez dostosowanie jej do się do schematu po stronie serwera. **Schematy są globalnie unikalne**. Jest to zagwarantowane przez [[OID]].

#### Przykład zastosowania:
Jeśli będziemy chcieli podać numer telefonu, schemat nie dopuści do sytuacji, że w jednym miejscu będzie to `phone`, a w innym `tel`, wymusi aby atrybut nazywał się tak jak zostało to ustalone w schemacie.

# Składowe schematu
---
- lista dostępnych klas
- wymagane atrybuty w poszczególnych klasach
- dozwolone atrybuty klas
- typy danych dla atrybutów
- metody przetwarzania atrybutów

# Klasy obiektów
---

Każdy węzeł jest instancją jakiejś klasy, może się zdarzyć, że więcej niż jednej. W takiej sytuacji atrybuty wymagane i dozwolone sumują się. Klasy mogą być dziedziczone, lecz nie mogą redefiniować atrybutów klasy bazowej.

#### Przykład definicji klasy
``` LDAP
objectclass ( 1.3.6.1.1.1.2.0
NAME 'posixAccount'
SUP top AUXILIARY
DESC 'Abstraction of an account with POSIX attributes'
MUST ( cn $ uid $ uidNumber $ gidNumber $ homeDirectory )
MAY ( userPassword $ loginShell $ gecos $ description ) )
```

### Typy klas w schemacie
#### Klasy strukturalne (_structural_)
>Definiują podstawową charakterystykę obiektu, cel stosowania tego obiektu. Obiekt musi
być wystąpieniem dokładnie jednej klasy strukturalnej.
**Przykłady**: `account`, `group`.

#### Klasy pomocnicze (_auxiliary_)
>Rozszerzają atrybuty klasy strukturalnej. Większość klas ma
charakter pomocniczy.
**Przykłady**: `posixAccount`, `posixGroup`.

#### Klasy abstrakcyjne (_abstract_)
>Wykorzystywane do definicji klas bazowych [[LDAP]], typów specjalnych.
**Przykłady**: `top` czy `alias`.

Na szczycie hierarchii dziedziczenia znajduje się klasa specjalna `top` z której wywodzą się wszystkie pozostałe klasy. Posiada ona atrybut wymagany `objectClass`, wskazujący na klasę której instancją jest dany obiekt.

> [!Box]- Obiekty wieloklasowe
> ![[Pasted image 20240525195113.png]]

# Atrybuty
---

Posiadane przez nie nazwy nie rozróżniają liter. Każdy z atrybutów jest identyfikowany przez [[OID]]. Dla każdego atrybutu mamy przypisany typ danych i operatory jakie obsługuje. Ostatnią własnością atrybutu jest to czy jest on wielowartościowy czy nie.

#### Przykład
| Nazwa skrócona | Nazwa pełna         | Opis                      |
| -------------- | ------------------- | ------------------------- |
| uid            | User Identifier     | Identyfikator użytkownika |
| cn             | Common Name         | Nazwa                     |
| sn             | Surename            | Nazwisko                  |
| ou             | Organizational Unit | Jednostka organizacyjna   |
| o              | Organization        | Jednostka                 |
| dc             | Domain Component    | Składnik nazwy domenowej  |
| c              | Country             | Państwo                   |

### Typy atrybutów
Podstawowe typy to liczby, wartości logiczne i tekstowe, jednak typem określamy też to w jaki sposób te podstawowe typy wykorzystujemy, stąd może być ich sporo. Zanim stworzymy jednak swój typ, warto sprawdzić czy taki typ nie był już przez kogoś zdefiniowany. Typy danych również są identyfikowane przez [[OID]]

#### Przykład
| Typ danych | OID | Opis |
| ---------- | --- | ---- |
|Boolean | 1.3.6.1.4.1.1466.115.121.1.7 | Wartość logiczna |
|Distinguished Name | 1.3.6.1.4.1.1466.115.121.1.12 | Nazwa wyróżniająca |
|Directory String | 1.3.6.1.4.1.1466.115.121.1.15 | Łańcuch tekstowy UTF‐8 |
|Integer | 1.3.6.1.4.1.1466.115.121.1.27 | Liczba całkowita |
|OID | 1.3.6.1.4.1.1466.115.121.1.38 | Identyfikator obiektu |


# Operatory
---
- porównania (_equality_)
- porządkowania (_ordering_)
- przeszukiwania (_substring_)
