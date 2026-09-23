---
tags:
  - obrona
up: "[[NPR 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 8
source: "slajdy–merged.pdf"
slajdy: "255–264"
---
# NPR 11. Przestrzeń krotek — Linda i JavaSpaces
---
> Ostatnie z omawianych podejść i najbardziej odmienne od pozostałych. W [[NPR 08 MOM i systemy kolejkowania komunikatów|MOM]] pośrednikiem jest **kolejka**, identyfikowana nazwą; tutaj — **przestrzeń krotek**, w której komunikat identyfikuje się **przez jego treść**. Wykład omawia model Lindy (Gelernter, 1985): pojęcia krotki i przestrzeni, cztery operacje dostępu, **dopasowanie asocjacyjne** i gwarancje żywotności, a następnie ich realizację w **JavaSpaces**.

---
## Koncepcja
<sub>slajdy–merged.pdf, slajd 256</sub>

- Mechanizm komunikacji międzyprocesowej zaproponowany przez **Davida Gelerntera w 1985**.
- **Luźne powiązanie** komunikujących się procesów — nie muszą znać się wzajemnie, **nie muszą działać jednocześnie** (komunikacja **nieustanna**).
- **Asocjacyjna identyfikacja komunikatów** (w odróżnieniu od kolejkowania) we **współdzielonej przestrzeni**.

Współczesne implementacje:
- **JavaSpaces** — Sun Microsystems (w ramach technologii **Jini**, projekt przejęty przez ASF — **Apache River**), komercyjna implementacja dostarczana przez **Gigaspaces**,
- **TSpaces** — IBM.

Dwa pierwsze punkty są wspólne z MOM — trzeci jest tym, co odróżnia. **Asocjacyjna** znaczy: odbiorca nie mówi „daj mi z kolejki X", tylko **opisuje, jak ma wyglądać** komunikat, który go interesuje. Przestrzeń krotek jest więc czymś pomiędzy kolejką a bazą danych zapytań przez wzorzec.

---
## Model
<sub>slajdy–merged.pdf, slajd 257</sub>

| Pojęcie | Definicja |
|---|---|
| **Krotka** | **uporządkowana kolekcja danych określonych typów — atrybutów**, przy czym atrybuty **mogą (ale nie muszą) mieć nadaną konkretną wartość** |
| **Przestrzeń krotek** | **wspólne miejsce dostępne dla kooperujących procesów**, gdzie gromadzone są krotki |

Interfejs dostępu do przestrzeni krotek:

| Operacja | Działanie |
|---|---|
| **`Output`** | **umieszczanie** krotki w przestrzeni |
| **`Input`** | **pobieranie** krotki z przestrzeni |
| **`Read`** | **odczytywanie krotki bez pobierania** — odczytana krotka w dalszym ciągu pozostaje w przestrzeni |
| **`Try_Input`, `Try_Read`** | **nieblokujące** wersje `Input` i `Read` |

Istnienie wariantów `Try_*` mówi, że `Input` i `Read` są **blokujące**: proces czekający na krotkę, której jeszcze nie ma, zostaje wstrzymany. To czyni z przestrzeni krotek mechanizm **synchronizacji**, a nie tylko komunikacji — na przestrzeni krotek można zaimplementować semafor, barierę czy pulę zadań.

Możliwość pozostawienia atrybutu **bez wartości** jest tu kluczowa — ten sam zapis krotki służy i jako **dane**, i jako **wzorzec zapytania**.

---
## Dopasowanie
<sub>slajdy–merged.pdf, slajd 258</sub>

Krotka staje się dostępna w przestrzeni po wykonaniu operacji `Output`, której parametrami są **wartości atrybutów**:

```
Output(4, 1, "Good morning")
```

Operacja `Input` powoduje **pobranie (usunięcie z przestrzeni)** krotki, której wartości atrybutów są **zgodne z parametrami operacji**:

```
Input(4, 1, text: String)
```

**Wartości pozostałych atrybutów zostaną nadane zgodnie z zawartością pobranej krotki** — tu zmienna `text` otrzyma `"Good morning"`.

Jeśli krotka została umieszczona w przestrzeni **bez podania wartości** któregoś z atrybutów:

```
Output(, 1, "Anybody there?")
```

to **może ona zostać pobrana przez `Input` z dowolną wartością tego atrybutu lub pominięciem tej wartości**.

Symetria jest pełna: **niewypełnione miejsce po stronie zapytania** oznacza „cokolwiek, i przypisz mi to", a **niewypełnione miejsce po stronie krotki** oznacza „pasuję do czegokolwiek". Dopasowanie działa więc w obie strony.

![[npr-lin-s259-dopasowanie-krotek.png]]
<sub>`Input(4, 1, text: String)` dopasowuje się zarówno do `[4, 1, "Good morning"]`, jak i do `[, 1, "Anybody there?"]`. Równoległe `Input(4, 2, text: String)` **czeka**, dopóki `Output` nie umieści `[4, 2, "How are you?"]`. slajdy–merged.pdf, slajd 259</sub>

---
## Własności przestrzeni
<sub>slajdy–merged.pdf, slajd 260</sub>

- **Przestrzeń krotek jest zbiorem (nie kolejką)** — krotki **nie są uporządkowane** i **mogą być odbierane w innej kolejności niż były umieszczane**.
- W realizacji operacji dostępu gwarantowana jest **ogólnie rozumiana żywotność**:
  - Jeśli procesy czekają (w operacji `Input`) na krotkę, to **przy odpowiednio dużej liczbie umieszczonych krotek** z oczekiwanymi wartościami atrybutów **każdy w końcu ją otrzyma**.
  - Jeśli krotka jest w przestrzeni, to **po odpowiednio dużej liczbie operacji `Input`** (ze zgodnymi parametrami) **zostanie w końcu odczytana**.

To najważniejsza różnica wobec kolejki i trzeba ją rozumieć ściśle. **Nie ma FIFO** — jeśli w przestrzeni leży kilka pasujących krotek, pobrana zostanie **dowolna**. W zamian dostajemy jedynie **żywotność**: ani proces, ani krotka nie będą czekać w nieskończoność. Jest to gwarancja typu „w końcu", bez żadnego ograniczenia na czas.

Konsekwencja praktyczna: kolejności trzeba pilnować samemu, kodując ją w atrybutach krotki — dokładnie tak, jak w przykładzie ze slajdu 259, gdzie drugi atrybut pełni rolę licznika.

---
## JavaSpaces
<sub>slajdy–merged.pdf, slajd 261</sub>

- **Odpowiednikiem krotki jest obiekt klasy implementującej interfejs `Entry`.**
- **Zmienne instancji obiektu-krotki muszą być publiczne.**
- Obiekt-krotka ma **ustalony czas życia** w przestrzeni — **czas wynajmowania przestrzeni** (_lease time_).
- **Czas życia można zwiększać nawet po umieszczeniu** obiektu-krotki w przestrzeni.
- Operacje realizowane są poprzez metody **`write`, `take`, `read`, `takeIfExists`, `readIfExists`** interfejsu `JavaSpace`.

Odwzorowanie operacji Lindy na JavaSpaces:

| Linda | JavaSpaces |
|---|---|
| `Output` | `write` |
| `Input` | `take` |
| `Read` | `read` |
| `Try_Input` | `takeIfExists` |
| `Try_Read` | `readIfExists` |

Wymóg **publicznych pól** wynika wprost z mechanizmu dopasowania: przestrzeń musi umieć porównać pola obiektu-wzorca z polami obiektów przechowywanych, a robi to przez refleksję.

**Dzierżawa** (_lease_) jest dodatkiem JavaSpaces, którego w modelu Lindy nie ma. Rozwiązuje problem, z którym model Lindy sobie nie radzi: krotka, po którą nikt nigdy nie przyjdzie, zostawałaby w przestrzeni na zawsze. Jest to ten sam pomysł, co **wygaśnięcie** przy usuwaniu osieroconych obliczeń — zob. [[NPR 01 Zdalne wywoływanie procedur (RPC)#Usuwanie osieroconych obliczeń|NPR 01]] — oraz **czas życia komunikatu** w [[NPR 10 JMS#Czas życia|JMS]].

### Definicja krotki
<sub>slajd 262</sub>

```java
public class Message implements Entry {
   public Integer id;
   public Integer num;
   public String  text;

   public Message() {}

   public Message(Integer i, Integer n, String t) {
      id   = i;
      num  = n;
      text = t;
   }
}
```

Pola są typów **`Integer` i `String`**, a nie `int` — konieczne, bo brak wartości atrybutu wyraża się przez **`null`**, którego typ prosty nie przyjmie. Bezparametrowy konstruktor jest wymagany, żeby przestrzeń mogła odtworzyć obiekt po stronie odbiorcy.

### Umieszczanie krotki
<sub>slajd 263</sub>

```java
JavaSpace space = getSpace();
msg = new Message(new Integer(4),
                  new Integer(counter++),
                  "Good morning");
Lease l = space.write(msg, null, 6*60*60*1000);
```

Trzeci argument to **czas dzierżawy** w milisekundach (tu 6 godzin); drugi — transakcja (`null` = brak). `write` zwraca obiekt `Lease`, przez który dzierżawę można później **przedłużyć**.

### Odczyt przez wzorzec
<sub>slajd 264</sub>

```java
JavaSpace space = getSpace();
template = new Message(new Integer(4),
                       null,
                       null);
Message msg = (Message) space.read(template,
                                   null,
                                   60*60*1000);
```

**Wzorzec jest obiektem tej samej klasy**, w którym pola nieistotne ustawiono na `null` — to bezpośrednie odwzorowanie zapisu `Input(4, …, …)` z Lindy. Trzeci argument `read` to **czas oczekiwania** (nie dzierżawy): operacja zablokuje się najwyżej na godzinę.

---
## Braki w prezentacjach

> [!todo] Czego nie ma na slajdach 255–264
> - **Operacja `eval`** — czwarta, obok `out`/`in`/`rd`, operacja oryginalnej Lindy, umieszczająca w przestrzeni krotkę **obliczaną współbieżnie**. To ona czyniła z Lindy język programowania równoległego, a nie tylko mechanizm komunikacji.
> - **Wzorce programistyczne** na przestrzeni krotek — pula zadań (_master-worker_), semafor, bariera; slajdy podają mechanizm bez ani jednego zastosowania.
> - **Realizacja przestrzeni** — czy jest scentralizowana, czy rozproszona, i jakim kosztem realizowane jest dopasowanie asocjacyjne.
> - **Transakcje w JavaSpaces** — drugi parametr `write`/`read` jest we wszystkich przykładach `null`, bez wyjaśnienia, czym jest.
> - **Porównanie z pamięcią współdzieloną (DSM)** — przestrzeń krotek jest wymieniana obok DSM jako wariant „dostępu do wspólnej przestrzeni" (zob. [[Narzędzia Przetwarzania Rozproszonego/Paradygmat Interakcji Pomiędzy Zdalnymi Jednostkami]]), ale wykład tego zestawienia nie robi; modele spójności pamięci współdzielonej są w [[RSO 02 Danocentryczne modele spójności]].
