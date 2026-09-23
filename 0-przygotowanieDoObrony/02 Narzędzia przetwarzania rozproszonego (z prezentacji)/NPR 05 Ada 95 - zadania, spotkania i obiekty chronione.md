---
tags:
  - obrona
up: "[[NPR 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 9
source: "slajdy–merged.pdf"
slajdy: "265–300"
---
# NPR 05. Ada 95 — zadania, spotkania i obiekty chronione
---
> Ada jest jedynym omawianym narzędziem, w którym **współbieżność jest wbudowana w język**, a nie dostarczona biblioteką. Wykład buduje dwa niezależne mechanizmy: **spotkania asymetryczne** (_rendez-vous_) do komunikacji **synchronicznej między parą zadań** oraz **obiekty chronione** do komunikacji **asynchronicznej między wieloma zadaniami**. Trzon to instrukcja `select` z czterema zastosowaniami i mechanizm **barier** w obiektach chronionych, spięty na końcu instrukcją `requeue`.

---
## Zadania
<sub>slajdy–merged.pdf, slajd 266</sub>

- **Zadania** (_tasks_) są **jednostkami strukturalizacji programów współbieżnych**.
- Zadanie jest **obiektem typu zadaniowego**. W programie można utworzyć **wiele zadań na podstawie tego samego typu zadaniowego**; wszystkie obiekty danego typu mają jednakową **specyfikację (interfejs)** i tę samą **implementację**.
- Obiekty zadaniowe są tworzone **tak jak inne obiekty języka**: mogą być deklarowane **statycznie** lub kreowane **dynamicznie**.

To pierwsza istotna różnica wobec Javy: zadanie nie jest instancją klasy bibliotecznej, którą się „uruchamia", tylko **bytem językowym z własną składnią deklaracji** — zadanie zadeklarowane w bloku startuje automatycznie razem z tym blokiem.

### Dwa mechanizmy komunikacji
<sub>slajd 267</sub>

| Mechanizm | Charakter | Uczestnicy |
|---|---|---|
| **spotkania asymetryczne** (_rendez-vous_) | komunikacja **synchroniczna** | **para** zadań |
| **obiekty chronione** | komunikacja **asynchroniczna** | **wiele** zadań |

Cała reszta wykładu to rozwinięcie tych dwóch wierszy.

---
## Specyfikacja i implementacja zadania
<sub>slajdy–merged.pdf, slajdy 268–269</sub>

Zadanie, jak każda jednostka programowa Ady, dzieli się na **specyfikację** (co widać z zewnątrz) i **treść** (jak to działa).

```ada
task Zad is
   ...
   entry E1(p1: in t1; p2: out t2);
   ...
private
   entry E2(...);
   ...
end Zad;
```

```ada
task body Zad is
   ...
   accept E1(p1: in t1; p2: out t2) do
      ...
   end E1;
   ...
   accept E2(...) do
      ...
   end E2;
   ...
end Zad;
```

**Wejście** (`entry`) jest tym, co w interfejsie zdalnym odpowiadałoby metodzie — z tą różnicą, że wywołanie wejścia **synchronizuje** obie strony. Wejścia zadeklarowane w części `private` nie są dostępne dla klientów z zewnątrz; ich rola ujawnia się przy [[#Rekolejkowanie|rekolejkowaniu]].

### Typ zadaniowy
<sub>slajdy 270–271</sub>

```ada
task type T is
   ...
   entry E1(...);
   ...
end T;
```

Skoro zadanie jest zwykłym obiektem języka, typ zadaniowy można użyć wszędzie tam, gdzie typ:

```ada
tablica : array(1..5) of T;

type R is
record
   Zadanie : T;
   ...
end record;
...
R1, R2 : R;
```

Deklaracja `tablica : array(1..5) of T` tworzy **pięć równolegle działających zadań** — bez żadnej pętli tworzącej wątki.

---
## Spotkania asymetryczne
<sub>slajdy–merged.pdf, slajd 272</sub>

W komunikacji między zadaniami z wykorzystaniem mechanizmu spotkań istotne jest **wyróżnienie roli**, jaką może w danej chwili pełnić zadanie — na tym polega **asymetria**.

| Rola | Kiedy |
|---|---|
| **bierne (serwer)** | jeśli **udostępnia lub jest gotowe udostępnić** usługi identyfikowane przez **nazwy wejść** |
| **czynne (klient)** | jeśli **wywołuje właśnie wejście** jakiegoś serwera |

Role są przypisane **chwilowo**, nie na stałe: to samo zadanie może w różnych momentach być klientem i serwerem. Asymetria polega na tym, że **klient musi znać nazwę serwera**, a serwer o kliencie nic nie wie — składnia wywołania jest jednostronna:

```ada
Nazwa_zadania.wejście(param, ...);   -- strona klienta, slajd 273

accept wejście(param: typ) do        -- strona serwera, slajd 274
   ...
end wejście;
```

### Przebieg spotkania
<sub>slajdy 275–276</sub>

Spotkanie jest **synchroniczne w obie strony**: ta strona, która dotrze pierwsza, czeka na drugą. Możliwe są więc dwa scenariusze.

**Serwer czeka** — `accept` osiągnięty przed wywołaniem klienta:

![[npr-ada-s275-spotkanie-oczekiwanie-serwera.png]]
<sub>slajdy–merged.pdf, slajd 275</sub>

**Klient czeka** — wywołanie wykonane przed osiągnięciem `accept`:

![[npr-ada-s276-spotkanie-oczekiwanie-klienta.png]]
<sub>slajdy–merged.pdf, slajd 276</sub>

W obu przypadkach **tylko ciało `accept … do … end`** jest wykonywane przy wstrzymanym kliencie — po `end` oba zadania biegną znowu równolegle. To w tym oknie przekazuje się parametry `out`.

---
## Instrukcja `select`
<sub>slajdy–merged.pdf, slajd 277</sub>

Jedna instrukcja, cztery zastosowania:

| Zastosowanie | Strona |
|---|---|
| **Oczekiwanie selektywne** | serwer |
| **Terminowe wywołanie wejścia** | klient |
| **Warunkowe wywołanie wejścia** | klient |
| **Asynchroniczna zmiana wątku sterowania** | dowolna |

---
## Oczekiwanie selektywne (strona serwera)
<sub>slajdy–merged.pdf, slajd 278</sub>

Umożliwia po stronie serwera:
- oczekiwanie na **więcej niż jedno spotkanie** — **alternatywa**,
- oczekiwanie na rozpoczęcie spotkania **w ustalonym odcinku czasu**,
- **wycofanie oferty** spotkania, jeżeli nie może ono nastąpić natychmiast,
- **zakończenie istnienia zadania**, jeżeli nie istnieją klienci, którzy wywołują jego wejścia.

Każdemu z czterech punktów odpowiada osobna konstrukcja: gałąź `or accept`, gałąź `delay`, gałąź `else`, gałąź `terminate`.

### Alternatywa
<sub>slajd 279</sub>

```ada
task body T is
begin
  loop
    select
       accept E1(...) do
         ...
       end;
    or
       accept E2(...) do
         ...
       end;
    end select;
  end loop;
end T;
```

### Alternatywa z gałęziami dozorowanymi
<sub>slajd 280</sub>

```ada
task body T is
begin
  loop
    select
       when i > 0 => accept E1(...) do
         ...
       end;
    or
       when i = 0 => accept E2(...) do
         ...
       end;
    end select;
  end loop;
end T;
```

### Obsługa kolejki żądań
<sub>slajd 281</sub>

- Przy **braku pragm** przyjmowana jest strategia **FIFO** obsługi kolejki żądań.
- Jeśli w momencie osiągnięcia instrukcji `select` zadania-klienci czekają **w kolejkach kilku wejść**, to **wybór kolejki jest niedeterministyczny**.

Dwa poziomy rozstrzygnięć: **w obrębie jednego wejścia** kolejność jest określona (FIFO), **między wejściami** — nieokreślona. Programista nie może więc opierać poprawności na tym, które wejście zostanie obsłużone pierwsze; do sterowania służą dozory.

### Dozory
<sub>slajdy 282–283</sub>

| Pojęcie | Definicja |
|---|---|
| **Dozór** (_guard_) | wyrażenie logiczne poprzedzające gałąź `accept` instrukcji `select` |
| **Gałąź otwarta** | gałąź, dla której dozór jest **spełniony** |
| **Gałąź zamknięta** | pozostałe |

Zasady wykonania:
- Gałęzie **bez dozorów** równoważne są gałęziom z dozorami `True`.
- Wykonanie instrukcji `select` **rozpoczyna się od obliczenia dozorów wszystkich gałęzi**.
- **Tylko gałęzie otwarte** brane są pod uwagę w trakcie dalszego wykonania instrukcji `select`.
- Wartości wszystkich dozorów obliczane są każdorazowo **tylko raz, na początku** wykonania instrukcji `select`.
- Jeśli **wszystkie** gałęzie chronione są dozorami, a ich wartości są równe `False`, to generowany jest wyjątek **`Program_Error`**.

Reguła „raz, na początku" jest praktycznie najważniejsza: dozór **nie jest** przeliczany, gdy w trakcie oczekiwania zmieni się stan zadania. Zmiana dozoru staje się widoczna dopiero przy następnym wykonaniu `select` — stąd typowy wzorzec `loop … select … end select; end loop`.

### Przeterminowanie spotkań
<sub>slajd 284</sub>

```ada
task body T is
begin
  loop
    select
       accept E1(...) do
         ...
       end;
    or
       delay 5.0;   -- możliwe również delay until
       exit;        -- wyjście z pętli
    end select;
  end loop;
end T;
```

### Gałąź `else`
<sub>slajd 285</sub>

```ada
task body T is
begin
  loop
    select
       accept E1(...) do
         ...
       end;
    else
       exit;   -- wyjście
    end select;
  end loop;
end T;
```

Gałąź `else` pozwala serwerowi **wycofać ofertę spotkania**, gdy brak jest zadań-klientów **już oczekujących** na spotkanie. Może wystąpić w instrukcji `select` **tylko raz** i **nie może być chroniona dozorem**.

Różnica wobec `delay 0.0` jest pojęciowa: `else` nie czeka **w ogóle**, sprawdza stan kolejek w chwili wykonania.

### Gałąź `terminate`
<sub>slajd 286</sub>

```ada
task body T is
begin
  loop
    select
       accept E1(...) do
         ...
       end;
    or
       terminate;
    end select;
  end loop;
end T;
```

Gałąź `terminate` jest wybierana, gdy **jednostka macierzysta zadania zakończyła się** i **wszystkie zadania potomne albo zakończyły się, albo gotowe są wybrać gałąź `terminate`**. Może być **poprzedzona dozorem**. **Nie może występować jednocześnie z gałęzią `delay` lub `else`**.

To rozwiązanie problemu, który w Javie rozwiązuje się wątkami-demonami: zadanie-serwer działające w nieskończonej pętli samo z siebie nigdy by się nie skończyło i blokowałoby zakończenie programu. `terminate` mówi: „skończ mnie, gdy nikt już nie może do mnie zadzwonić".

Zakaz łączenia z `delay`/`else` wynika z tego, że każda z tych gałęzi **wyklucza nieograniczone czekanie**, a `terminate` wymaga właśnie stanu biernego oczekiwania.

---
## Terminowe i warunkowe wywołanie wejścia (strona klienta)
<sub>slajdy–merged.pdf, slajdy 287–288</sub>

**Terminowe wywołanie wejścia** umożliwia po stronie klienta **oczekiwanie na rozpoczęcie spotkania w ustalonym odcinku czasu**:

```ada
select
  server.E1(...);
  -- tu mogą być jeszcze jakieś instrukcje
or
  delay 2.0;   -- ewent. delay until
  -- tu mogą być jeszcze jakieś instrukcje
end select;
```

**Warunkowe wywołanie wejścia** umożliwia po stronie klienta **wycofanie oferty spotkania, jeżeli nie może ono nastąpić natychmiast**:

```ada
select
  server.E1(...);
  -- tu mogą być jeszcze jakieś instrukcje
else
  -- tu mogą być jeszcze jakieś instrukcje
end select;
```

Symetria jest pełna: `delay` i `else` znaczą u klienta to samo, co u serwera — z ograniczonym czekaniem i bez czekania w ogóle. Warunkiem powodzenia jest **rozpoczęcie** spotkania, nie jego zakończenie: gdy `accept` już się zaczął, klient czeka do końca niezależnie od `delay`.

---
## Asynchroniczna zmiana wątku sterowania
<sub>slajdy–merged.pdf, slajd 289</sub>

Umożliwia **przerwanie wykonywania programu** po zakończeniu **instrukcji wyzwalającej** (po ustalonym czasie lub zakończeniu wywołania wejścia itp.):

```ada
select
  delay 5.0;
  Put_line("Czas minął");
then abort
  obliczaj(...);
end select;
```

Konstrukcja `select … then abort` odwraca zwykły porządek: to część **po `then abort`** jest właściwym obliczeniem, a część przed nią — **wyzwalaczem**, który je przerywa. Jeśli `obliczaj` skończy się przed upływem 5 sekund, `delay` jest anulowane i komunikat się nie pojawia. To jedyny w Adzie sposób na **wywłaszczenie trwającego obliczenia** bez współpracy ze strony tego obliczenia.

---
## Obiekty chronione
<sub>slajdy–merged.pdf, slajd 290</sub>

- **Obiekt chroniony** jest jednostką programową, która **organizuje dostęp zadań do grupowanych przez siebie danych współdzielonych**.
- Budowa obiektu chronionego jest **podobna do budowy pakietu i zadania** — składa się ze **specyfikacji** i **treści** implementującej obiekt.
- Możliwe jest definiowanie **typów chronionych**.

Obiekt chroniony to **monitor** wbudowany w język: dane są niedostępne inaczej niż przez zadeklarowane operacje, a wzajemne wykluczanie zapewnia kompilator i środowisko wykonawcze, nie programista.

### Struktura
<sub>slajd 291</sub>

- Specyfikacja obiektu (oraz typu) chronionego **zawsze zawiera część publiczną i część prywatną**.
- **Część publiczną** tworzą deklaracje **funkcji, procedur oraz wejść**.
- W **części prywatnej** występują deklaracje **zmiennych współdzielonych** i — opcjonalnie — deklaracje **wewnętrznych** funkcji, procedur i wejść.

### Dostęp
<sub>slajd 292</sub>

- Dostęp do obiektu chronionego jest możliwy **tylko poprzez wywołania funkcji, procedur i wejść publicznych** i odbywa się **zgodnie z zasadą wzajemnego wykluczania**.
- **Funkcje** pozwalają **tylko na odczyt** danych współdzielonych (podanych w części `private`), a **procedury i wejścia** — na ich **modyfikowanie**.

To rozróżnienie nie jest konwencją — **kompilator je egzekwuje** i na nim opiera dobór blokady.

### Blokady
<sub>slajd 293</sub>

W momencie, gdy zadanie wywołuje wejście lub procedurę obiektu chronionego, ten może być **zajęty obsługą innego wywołania** (zablokowany). Z każdym obiektem chronionym związane są **dwie blokady**:

| Blokada | Aktywna, gdy |
|---|---|
| **do czytania** (_shared read lock_) | obiekt chroniony obsługuje wywołanie swojej **funkcji** |
| **do pisania** (_exclusive read/write lock_) | obiekt obsługuje wywołanie **procedury lub wejścia** |

Jest to więc klasyczny **zamek czytelników i pisarzy** — odpowiednik `ReadWriteLock` z [[NPR 07 Pakiet java.util.concurrent|java.util.concurrent]], z tą różnicą, że wybór blokady jest **automatyczny**, wynikający z rodzaju wywołanej operacji.

### Zasady synchronizacji dostępu
<sub>slajdy 294–295</sub>

1. Jeżeli obiekt chroniony ma założoną blokadę **read** i wywoływana jest jego **funkcja**, to funkcja ta **zostaje wykonana**.
2. Jeżeli obiekt chroniony ma założoną blokadę **read** i wywoływane jest jego **wejście lub procedura**, to wywołanie jest **opóźniane**, dopóki są zadania aktywne wewnątrz obiektu chronionego.
3. Jeżeli obiekt chroniony ma założoną blokadę **read/write**, to wywołanie jest **opóźniane**, dopóki są zadania aktywne wewnątrz obiektu chronionego.
4. Jeżeli nadchodzi kolej wykonania wywoływanego **wejścia** obiektu chronionego, lecz **bariera ma wartość `False`**, to wywołanie jest **ustawiane w kolejce związanej z tą barierą**, czekając na spełnienie warunku bariery; **obiekt nie zostaje jeszcze zablokowany**.

Punkt 4 jest najważniejszy i odróżnia wejścia od procedur: zadanie czekające na barierę **nie trzyma blokady**, więc inne zadania mogą wejść i zmienić stan tak, żeby bariera się otworzyła. Bez tej własności każdy bufor ograniczony natychmiast by się zakleszczał.

### Przykład — zmienna chroniona
<sub>slajdy 296–297</sub>

```ada
protected zmienna_chroniona is
   function czytaj return zapis;
   procedure pisz (x: in zapis);
private
   element : zapis;   -- zmienna współdzielona
end zmienna_chroniona;
```

```ada
protected body zmienna_chroniona is
   function czytaj return zapis is
   begin
      return element;
   end czytaj;

   procedure pisz (x: in zapis) is
   begin
      element := x;
   end pisz;
end zmienna_chroniona;
```

Sam ten przykład nie potrzebuje wejść — nie ma warunku, na który trzeba czekać. Wejścia stają się konieczne dopiero przy buforze.

---
## Wejścia obiektu chronionego i bariery
<sub>slajdy–merged.pdf, slajd 298</sub>

- Wejścia, **podobnie jak procedury**, umożliwiają **modyfikację** obiektu.
- W części implementacyjnej z każdym z zadeklarowanych wejść jest związany **warunek wykonania wejścia**, nazywany **barierą** (_barrier_).
- **Treść wołanego wejścia jest wykonywana tylko, jeśli warunek bariery jest spełniony.**

Bariera obiektu chronionego pełni tę samą rolę, co **zmienna warunkowa** monitora (`wait`/`notify` w Javie), ale jest **deklaratywna**: programista zapisuje *warunek*, a nie *gdzie zasnąć i kogo obudzić*. Nie da się więc zgubić sygnału ani zapomnieć o `notifyAll` — por. [[NPR 06 Wątki w Javie i elementarna synchronizacja#Zgubiony sygnał|NPR 06]].

### Przykład — bufor cykliczny
<sub>slajd 299</sub>

```ada
subtype Rozmiar is Integer range 1..Rozmiar_MAX;

protected type Bufor_cykliczny(N: Rozmiar := 100) is
    entry Put(x: in  Wartość);
    entry Get(x: out Wartość);
private
    bufor   : array(0..N-1) of Wartość;   -- bufor N-elementowy
    put_ptr : Integer := 0;               -- indeks miejsca wstawienia
    get_ptr : Integer := 0;               -- indeks miejsca pobrania
    licznik : Integer range 0..N := 0;    -- zajętość bufora
end Bufor_cykliczny;
```

`Put` i `Get` są **wejściami**, a nie procedurami, bo każde ma warunek: `Put` wymaga `licznik < N`, `Get` wymaga `licznik > 0`. Typ chroniony jest **parametryzowany** (`N: Rozmiar := 100`), więc z jednej deklaracji można tworzyć bufory różnych rozmiarów.

---
## Rekolejkowanie
<sub>slajdy–merged.pdf, slajd 300</sub>

**Żądanie wejścia może zostać przekazane do kolejki związanej z innym wejściem** (zadeklarowanym np. w części prywatnej). Instrukcja `requeue` może pojawić się **w obsłudze wejścia zadania lub obiektu chronionego**:

```ada
requeue E1;
```

`requeue` rozwiązuje sytuację, w której warunek dopuszczenia da się sprawdzić **dopiero po wejściu** do obiektu — na przykład gdy zależy od wartości parametru wywołania, której bariera nie widzi. Zadanie jest wtedy wpuszczane, a następnie **przekierowywane do kolejki innego wejścia** zamiast odrzucane; z punktu widzenia klienta jego jedno wywołanie nadal trwa. Docelowe wejście deklaruje się zwykle jako **prywatne**, bo nie jest przeznaczone do wołania z zewnątrz.

---
## Braki w prezentacjach

> [!todo] Czego nie ma na slajdach 265–300
> - **Rozproszona Ada** — pakiet Annex E, partycje, `pragma Remote_Call_Interface`, PolyORB. Zagadnienie 8 wymienia Adę wśród podejść do budowy systemów **rozproszonych**, a prezentacja omawia wyłącznie współbieżność **lokalną**. Opracowanie wspomina PolyORB jedną linijką bez wyjaśnienia. <sub>Opracowanie.pdf, s. 62</sub>
> - **Pragmy sterujące kolejkowaniem** — slajd 281 mówi o „braku pragm", nie podając, jakie pragmy istnieją (`Queuing_Policy`, `Priority_Queuing`).
> - **`'Count` i inne atrybuty wejść** — nieodzowne przy implementacji barier i semaforów, nie występują na slajdach.
> - **Priorytety zadań i dziedziczenie priorytetów** (`pragma Priority`, protokół pułapu) — brak.
> - **Pełna semantyka `abort` i zadań przerwanych** — slajd 289 pokazuje `then abort`, ale nie mówi, co się dzieje z zasobami przerwanego obliczenia.
> - **Porównanie z mechanizmami Javy** — zestawienie znajduje się w [[NPR 09 Wielozadaniowość i synchronizacja zadań i wątków|skompresowane/NPR 09]].
> - Sekcja „Współbieżność" w [[Narzędzia Przetwarzania Rozproszonego/ADA-95]] jest **pusta** — niniejsza notatka ją zastępuje; tamta notatka pozostaje przydatna jako opis **składni podstawowej** języka (typy, pętle, przekazywanie argumentów), której slajdy nie obejmują.
