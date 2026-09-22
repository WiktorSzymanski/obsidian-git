---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 46
---
# 46. Programowanie rozproszone z gwarancjami ostatecznej spójności – bezkonfliktowe typy replikowane (CRDT) i typy chmurowe
---
> W systemach **geo-replikowanych** i działających **offline** nie można czekać na synchronizację przy każdej operacji: CAP – przy podziale sieci trzeba wybrać dostępność. Zamiast tego każda replika **przyjmuje operacje lokalnie** i propaguje je asynchronicznie, a system gwarantuje **ostateczną spójność**. **CRDT** i **typy chmurowe** to dwa podejścia, w których **zbieżność replik wynika z konstrukcji typu danych**, a nie z ręcznego rozwiązywania konfliktów przez programistę.

## Modele spójności
- **Spójność ostateczna** (_Eventual Consistency_, W. Vogels): jeśli nie są wykonywane nowe aktualizacje, to **w końcu** wszystkie repliki zwrócą tę samą wartość.
  - słaba: nie mówi, **jaka** to będzie wartość ani **kiedy**; w międzyczasie możliwe **konflikty**, wymagające rozwiązywania (ostatni zapis wygrywa, wycofywanie, scalanie ręczne),
  - nie jest własnością bezpieczeństwa – każdy stan pośredni jest „dozwolony”.
- **Silna spójność ostateczna** (_Strong Eventual Consistency_, SEC – Shapiro, Preguiça, Baquero, Zawirski 2011):
  - **ostateczne dostarczenie** (_eventual delivery_) – aktualizacja dostarczona do jednej poprawnej repliki trafi w końcu do wszystkich,
  - **zbieżność** – repliki, które otrzymały **ten sam zbiór aktualizacji**, mają **równoważny stan** (niezależnie od kolejności),
  - **terminacja** – operacje kończą się lokalnie.
  - brak konfliktów do rozwiązywania, brak wycofań; odporność na $N-1$ awarii (replika działa samodzielnie).
- **Spójność silna** (linearyzowalność – [[Linearizability]]) wymaga koordynacji (konsensus) → niedostępna przy partycjach.
- Porównanie modeli: [[02 Danocentryczne modele spójności]], [[03 Modele spójności zorientowane na klienta]].

---
## CRDT – Conflict-free Replicated Data Types
> **Bezkonfliktowe replikowane typy danych** – abstrakcyjne typy danych (liczniki, zbiory, rejestry, mapy, listy, grafy), zaprojektowane tak, że **współbieżne aktualizacje** na różnych replikach **zawsze się scalają** do tego samego wyniku bez koordynacji → gwarantują **SEC**.

Dwie równoważne rodziny (każdą można emulować drugą):

### 1. CvRDT – oparte na stanie (_state-based, Convergent_)
Repliki okresowo przesyłają **cały stan**, a odbiorca **scala** go z własnym funkcją $merge$.

**Warunek zbieżności**: zbiór stanów tworzy **półkratę górną** (_join-semilattice_) z częściowym porządkiem $\leq$, gdzie:
- $merge(a, b) = a \sqcup b$ – **najmniejsze ograniczenie górne** (LUB), więc merge jest:
  - **łączny**: $(a \sqcup b) \sqcup c = a \sqcup (b \sqcup c)$,
  - **przemienny**: $a \sqcup b = b \sqcup a$,
  - **idempotentny**: $a \sqcup a = a$,
- każda aktualizacja jest **inflacyjna** (monotoniczna): $s \leq update(s)$.

**Zalety**: minimalne wymagania wobec komunikacji – komunikaty mogą ginąć, dublować się i przychodzić w dowolnej kolejności (wystarczy plotkowanie – [[Algorytmy Rozproszone/Gossiping]], anty-entropia). **Wada**: przesyłanie pełnego stanu. Rozwiązanie: **δ-CRDT** (_delta-state_, Almeida i in.) – przesyłanie tylko **delt** (fragmentów stanu zmienionych przez operacje), które także są elementami półkraty.

### 2. CmRDT – oparte na operacjach (_operation-based, Commutative_)
Repliki propagują **operacje**. Każda operacja ma dwie fazy:
- **prepare** (u źródła, bez efektów ubocznych) – przygotowanie argumentów na podstawie lokalnego stanu (np. wygenerowanie unikalnego znacznika),
- **effect** (_downstream_, na wszystkich replikach) – zastosowanie do stanu.

**Warunki zbieżności**:
- operacje dostarczane **dokładnie raz** i w porządku **przyczynowym** (_reliable causal broadcast_ – [[01 Komunikacja grupowa#Porządki dostarczania]]),
- operacje **współbieżne** (nieporównywalne przyczynowo) są **przemienne** (ich efekty można stosować w dowolnej kolejności).

**Zalety**: małe komunikaty. **Wady**: silniejsze wymagania wobec warstwy komunikacji (brak utraty i duplikatów, porządek przyczynowy – zegary wektorowe).

## Przykłady CRDT
### Licznik rosnący – G-Counter (state-based)
```
stan:     P : wektor [0..n-1] liczb naturalnych, początkowo zera
inc():    P[moja_replika] += 1
value():  Σ_i P[i]
merge(X, Y): Z[i] = max(X[i], Y[i])  dla każdego i
porządek: X ≤ Y  ⇔  ∀i X[i] ≤ Y[i]
```
Replika A: `[3,0,0]`, replika B: `[1,2,0]` → merge `[3,2,0]`, wartość 5. Maksimum po składowych jest łączne, przemienne i idempotentne – ponowne scalenie nic nie zmienia.

### Licznik PN-Counter
Dwa G-Countery: $P$ (inkrementacje) i $N$ (dekrementacje); $value = \sum P - \sum N$; merge – składowo dla obu. (Nie da się zrobić licznika z wektorem wartości i max, bo dekrementacja nie jest inflacyjna.)

**Op-based licznik**: operacje `inc`/`dec` są przemienne – wystarczy zastosować każdą raz.

### G-Set (zbiór tylko z dodawaniem)
`add(e)`: $A := A \cup \{e\}$; `lookup(e)`: $e \in A$; merge: suma zbiorów.

### 2P-Set (zbiór dwufazowy)
- dwa G-Sety: $A$ (dodane), $R$ (usunięte – **nagrobki**, _tombstones_),
- `add(e)`: $A \cup= \{e\}$; `remove(e)`: jeśli $e \in A$ → $R \cup= \{e\}$,
- `lookup(e)`: $e \in A \wedge e \notin R$,
- wada: **elementu raz usuniętego nie można ponownie dodać**; nagrobki rosną.

### LWW-Register (Last-Writer-Wins)
- stan: $(wartość, znacznik)$ – znacznik z zegara (fizycznego/hybrydowego HLC lub Lamporta) z identyfikatorem repliki do rozstrzygania remisów,
- `assign(v)`: $(v, now())$,
- merge: stan z **większym** znacznikiem.
- Proste, ale **współbieżne zapisy są tracone** (wygrywa jeden); zależne od jakości zegarów.

### MV-Register (Multi-Value)
Przechowuje **zbiór wartości współbieżnych** z wektorami wersji; odczyt może zwrócić kilka wartości (jak koszyk w Dynamo) – aplikacja lub następny zapis scala. Zapis zastępuje wszystkie wartości, które widział (przyczynowo wcześniejsze).

### LWW-Element-Set
Każdy element ma znacznik ostatniego dodania i ostatniego usunięcia; element należy do zbioru, jeśli znacznik dodania > znacznik usunięcia (z polityką dla remisu: _add-wins_ / _remove-wins_).

### OR-Set (Observed-Remove Set)
Rozwiązuje problemy 2P-Set i pozwala ponownie dodawać elementy.
```
stan:  S – zbiór par (e, u), u – unikalny znacznik (np. UUID/(replika, licznik))
add(e):      prepare: u := unikalny()           effect: S := S ∪ {(e, u)}
remove(e):   prepare: R := {(e,u) ∈ S}          effect: S := S \ R      // usuwa tylko OBSERWOWANE znaczniki
lookup(e):   ∃u: (e, u) ∈ S
```
- **Współbieżne `add(e)` i `remove(e)`** – remove usuwa tylko pary, które widział u źródła, a nowy znacznik dodania przetrwa → **add wygrywa** (_add-wins_).
- Wersja state-based przechowuje nagrobki usuniętych znaczników; optymalizacje z wektorami wersji (Optimized OR-Set, ORSWOT – bez nagrobków).

### Inne
- **Mapy CRDT** (klucze → zagnieżdżone CRDT, np. OR-Map z rekurencyjnym scalaniem),
- **Sekwencje / tekst** do wspólnej edycji: **RGA** (_Replicated Growable Array_ – elementy z unikalnymi identyfikatorami i odniesieniem do poprzednika, nagrobki), **Logoot**/**LSEQ** (gęste identyfikatory pozycji), **Treedoc**, **YATA** (Yjs), **Fugue**,
- **grafy** (2P2P-Graph), **flagi** (enable-wins), **liczniki ograniczone** (wymagają koordynacji przy granicy – _escrow_).

### Nagrobki i odśmiecanie
Wiele CRDT wymaga metadanych (nagrobki, wektory wersji), które rosną. Ich usuwanie wymaga wiedzy, że **wszystkie repliki** otrzymały daną operację (**stabilność przyczynowa**), co wymaga pewnej koordynacji lub kompromisu.

### Ograniczenia CRDT
- Nie każdą semantykę da się wyrazić bez koordynacji – **niezmienniki globalne** (saldo ≥ 0, unikalność nazw użytkowników, ograniczona liczba miejsc) wymagają synchronizacji (**zgodność I-confluence** – Bailis: operacja nie wymaga koordynacji ⇔ scalenie dwóch stanów spełniających niezmiennik też go spełnia).
- Semantyka scalania może być zaskakująca dla użytkownika (add-wins, przeplot tekstu).
- Narzut pamięci metadanych.

### Zastosowania
- **Riak** (liczniki, zbiory, mapy – riak_dt), **Redis Enterprise** Active-Active (CRDB), **Azure Cosmos DB** (multi-master), **AntidoteDB** (baza z transakcjami i CRDT – projekt SyncFree), **Akka Distributed Data**, **SoundCloud Roshi** (LWW-Set dla strumieni),
- **aplikacje local-first i współpraca w czasie rzeczywistym**: **Automerge**, **Yjs** (edytory, Jupyter), **Figma** (inspirowane CRDT), Apple Notes, Zed,
- sieci mobilne i IoT (praca offline, synchronizacja po powrocie).

---
## Typy chmurowe (_Cloud Types_)
S. **Burckhardt**, M. Fähndrich, D. Leijen, B. P. Wood (Microsoft Research) – „Cloud Types for Eventual Consistency” (**ECOOP 2012**); zrealizowane w środowisku **TouchDevelop** (programowanie aplikacji mobilnych na telefonie).

> **Typy chmurowe** to zestaw typów danych i prosty model programowania dla aplikacji mobilnych z danymi przechowywanymi **w chmurze**. Programista pisze **zwykły, sekwencyjny kod** na lokalnej kopii danych. Synchronizacja z chmurą odbywa się **w jawnych punktach** (`yield`), a sposób scalania współbieżnych zmian wynika z **wybranego typu** danych. Programista nie musi programować równolegle ani rozwiązywać konfliktów. Zob. [[Cloud Types]].

### Model programistyczny
- Każde urządzenie (klient) ma **lokalną replikę** danych chmurowych – działa **offline** i bez opóźnień.
- **`yield`** – **nieblokująca** synchronizacja: w tym punkcie (i tylko w nim – między kolejnymi fragmentami kodu) system **może** wysłać lokalne zmiany do serwera i/lub wprowadzić do lokalnej kopii zmiany innych klientów. Jeśli nie ma połączenia, nic się nie dzieje i kod idzie dalej. Kod między dwoma `yield` widzi **stabilny stan** (jak transakcja).
  - W TouchDevelop pętla zdarzeń robi `yield` **niejawnie** między obsługą zdarzeń (np. po każdym dotknięciu przycisku).
- **`flush`** – **blokująca** synchronizacja: czeka, aż wszystkie lokalne zmiany zostaną **zatwierdzone na serwerze** i stan lokalny odzwierciedla **stan serwera** → w tym punkcie program ma **silną spójność**. Wymaga połączenia; stosowany rzadko, dla operacji krytycznych.

### Model rewizji (_revision diagrams_)
Formalna semantyka oparta na **rewizjach** (jak gałęzie w systemie kontroli wersji):
- **serwer** utrzymuje **główną rewizję** (_main revision_),
- każdy klient pracuje na **rozgałęzieniu** (_fork_) z pewnego stanu serwera,
- przy synchronizacji lokalna rewizja jest **scalana** (_join_) z główną: zmiany klienta **odtwarzane** w kolejności zatwierdzania na serwerze, a następnie klient dostaje nowe rozgałęzienie od aktualnego stanu serwera,
- **diagram rewizji** – graf operacji fork/join; semantyka scalania określona dla każdej operacji typu chmurowego (**deterministyczna**, niezależna od opóźnień sieci).

Model gwarantuje **ostateczną spójność**: wszyscy klienci po synchronizacji widzą ten sam stan serwera, a wynik scalenia zależy tylko od **kolejności zatwierdzeń** na serwerze.

### Typy chmurowe
**Typy bazowe** (proste wartości):
| Typ | Operacje | Semantyka scalania |
|---|---|---|
| **`CInt`** (liczba) | `get()`, `set(v)`, **`add(d)`** | `add` jest **przemienne** – współbieżne dodawania **sumują się** (licznik); `set` nadpisuje. Przy scalaniu operacje klienta są odtwarzane: `add(d)` dodaje $d$ do bieżącej wartości serwera, a nie do wartości widzianej przy rozgałęzieniu |
| **`CString`** (napis) | `get()`, `set(s)`, **`setIfEmpty(s)`** | `set` – **ostatni zatwierdzony zapis wygrywa**; `setIfEmpty` – ustawia tylko, jeśli **w chwili scalania na serwerze** wartość jest pusta → wygrywa **pierwszy** zatwierdzony (bezpieczna rezerwacja) |
| `CBool`, `CDateTime`, … | analogicznie | – |

**Typy złożone** (struktura danych chmury):
| Typ | Opis | Semantyka |
|---|---|---|
| **`CArray<K, T>`** (tablica/indeks) | nieskończony „słownik” indeksowany kluczami (np. `CArray<string, CInt>`); wpisy **niejawnie istnieją** z wartością domyślną | brak konfliktów tworzenia – dwa klienty zapisujące do tego samego klucza scalają operacje na elemencie wg typu elementu |
| **`CSet<T>`** | zbiór wartości | dodawanie/usuwanie |
| **`CEntity`** (encja / tabela) | obiekty tworzone jawnie (`new`), mogą być **usuwane** (`delete`), odwołania między encjami, iteracja `entries` | tworzenie daje **unikalną** tożsamość (brak konfliktu); **usunięcie wygrywa** nad współbieżnymi aktualizacjami (_delete-wins_); usunięcie encji usuwa encje od niej zależne |

### Przykład – rezerwacja miejsca
```csharp
// każdy klient (np. aplikacja w telefonie), dane chmurowe:
//   Seat: CEntity { assignedTo: CString }

// wariant ostatecznie spójny (optymistyczny):
seat.assignedTo.setIfEmpty(customer);   // lokalnie natychmiast
yield;                                   // kiedyś zsynchronizuje się z serwerem
// UI może pokazać "zarezerwowano", ale przy konflikcie wygra pierwszy zatwierdzony na serwerze
// – po kolejnej synchronizacji klient zobaczy prawdziwego właściciela

// wariant z silną spójnością (operacja krytyczna):
seat.assignedTo.setIfEmpty(customer);
flush;                                    // czekaj na zatwierdzenie na serwerze
if (seat.assignedTo.get() != customer)
    print("reservation failed");          // wiadomo na pewno
```
**Licznik głosów/kliknięć**:
```csharp
votes[candidate].add(1);   // CArray<string, CInt> – dwa telefony offline dodają po 1 → po synchronizacji +2
```
Gdyby użyć `set(get() + 1)`, jedna aktualizacja zostałaby utracona (ostatni zapis wygrywa) – wybór **typu i operacji** określa semantykę.

### Implementacja
- Klient przechowuje **stan ostatnio znany z serwera** + **log lokalnych operacji** (niezatwierdzonych); stan lokalny = stan serwera z nałożonymi operacjami z logu.
- Synchronizacja: klient wysyła **operacje** (a nie stan) → serwer stosuje je w kolejności odbioru (daje to globalny porządek) → klient pobiera nowy stan / potwierdzenie i usuwa zatwierdzone operacje z logu.
- Kontynuacja pracy: **Global Sequence Protocol** (GSP, Burckhardt i in. 2015) – formalny model rozproszonego protokołu z globalnym porządkiem aktualizacji (sekwencja w chmurze), wraz z implementacją i dowodami poprawności.

### Typy chmurowe a CRDT
| | CRDT | Typy chmurowe |
|---|---|---|
| Architektura | dowolna (P2P, wiele serwerów, bez centrum) | **klient–serwer** (chmura jako źródło porządku) |
| Porządek aktualizacji | brak globalnego – zbieżność z przemienności / półkraty | **globalny porządek** zatwierdzeń na serwerze |
| Wymagania wobec operacji | przemienność operacji współbieżnych / merge = LUB | **dowolne deterministyczne** operacje (np. `setIfEmpty`, `set` zależne od kolejności) |
| Silna spójność | niedostępna (bez dodatkowej koordynacji) | dostępna na żądanie przez **`flush`** |
| Offline | tak | tak |
| Model programisty | wybór typu CRDT, zwykłe operacje | kod **sekwencyjny** + `yield`/`flush` + wybór typu chmurowego |
| Metadane | nagrobki, wektory wersji | log operacji klienta |
| Przykłady | Riak, Redis CRDB, Automerge, Yjs | TouchDevelop; podobne idee: Firebase, Realm Sync, Replicache |

**Wspólne**: gwarancja **ostatecznej spójności z konstrukcji typu**, brak jawnego rozwiązywania konfliktów, dostępność i praca offline, semantyka scalania określona przez **typ danych**.

## Zobacz też
- [[51 Big Data, NoSQL, CAP i PACELC - uzupełnienie]] – CAP, PACELC, Dynamo
- [[Magisterka/Architektura Aplikacji]] – eventual consistency w kontekście pracy magisterskiej
