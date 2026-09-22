---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 45
---
# 45. Modele obliczeń współbieżnych – model aktorów i rachunek pi
---
> **Modele obliczeń współbieżnych** to formalne lub półformalne opisy tego, jak niezależne jednostki obliczeniowe **wykonują się współbieżnie i komunikują**. Są współbieżnym odpowiednikiem maszyny Turinga i rachunku lambda. Dwa ważne modele oparte na **przekazywaniu komunikatów** (a nie pamięci współdzielonej) to **model aktorów** i **rachunek pi**.

Inne modele: sieci Petriego, CSP (Hoare), CCS (Milner), algebry procesów, PRAM, BSP, DAG obliczeń ([[44 Programowanie równoległe w Cilk]]).

---
## Model aktorów
Carl **Hewitt**, P. Bishop, R. Steiger (1973); rozwinięty semantycznie przez **Gula Aghę** (1986) i I. Greifa.

### Aktor
> **Aktor** to podstawowa jednostka obliczeń: ma **prywatny stan**, **adres** (tożsamość) i **skrzynkę pocztową** (_mailbox_). Komunikuje się z innymi aktorami **wyłącznie przez asynchroniczne komunikaty** wysyłane na adresy. „Wszystko jest aktorem”.

### Zachowanie – trzy aksjomaty
W odpowiedzi na odebrany komunikat aktor może (współbieżnie, w skończonej liczbie):
1. **wysłać** skończoną liczbę komunikatów do znanych sobie aktorów (adresy, które otrzymał w komunikatach, utworzył lub znał od początku),
2. **utworzyć** skończoną liczbę nowych aktorów,
3. **wyznaczyć zachowanie** (_become_) dla **następnego** komunikatu – czyli zmienić swój stan.

Nie ma innych sposobów interakcji: brak pamięci współdzielonej, brak blokad, brak synchronicznych wywołań.

### Własności
- **Asynchroniczność** – `send` nie blokuje nadawcy; komunikat trafia do skrzynki odbiorcy.
- **Sekwencyjne przetwarzanie komunikatów** w obrębie jednego aktora – aktor przetwarza **jeden komunikat naraz**, więc stan aktora nie wymaga synchronizacji (**brak wyścigów danych** na stanie aktora), a **współbieżność jest między aktorami**.
- **Brak gwarancji kolejności** dostarczenia w modelu teoretycznym (dostarczenie **gwarantowane**, ale opóźnienie nieograniczone – _unbounded nondeterminism_). W praktyce: Akka/Erlang zachowują kolejność FIFO **między parą** nadawca–odbiorca.
- **Przezroczystość położenia** (_location transparency_) – adres nie mówi, czy aktor jest lokalny, czy na innej maszynie → model naturalnie **rozproszony**.
- **Dynamiczna topologia** – nowe adresy przekazywane w komunikatach zmieniają, kto z kim może rozmawiać (mobilność – jak w rachunku pi).
- **Hermetyzacja** – stan zmieniany tylko przez sam aktor.
- **Tolerowanie awarii** (w Erlang/Akka): hierarchia **nadzoru** (_supervision_) – aktor-rodzic nadzoruje dzieci i przy awarii decyduje: restart, zatrzymanie, eskalacja; strategie **one-for-one** (restart tylko uszkodzonego) / **one-for-all** (restart wszystkich dzieci). Filozofia **„let it crash”** – zamiast defensywnej obsługi błędów, szybka awaria i restart w znanym stanie.

### Implementacje
**Erlang/OTP** (Ericsson, 1986 – centrale telefoniczne, wysoka dostępność, „dziewięć dziewiątek”), **Elixir**, **Akka** (Scala/Java, JVM), **Orleans** (.NET, „wirtualni aktorzy”), **Pony**, **CAF** (C++), Microsoft Dapr actors, Ray (Python).

#### Erlang
```erlang
-module(counter).
-export([start/0, loop/1]).

start() -> spawn(counter, loop, [0]).          % utworzenie aktora (procesu)

loop(N) ->
    receive                                     % selektywny odbiór z dopasowaniem wzorców
        {inc, From} ->
            From ! {ok, N + 1},                 % wysłanie komunikatu (!)
            loop(N + 1);                        % "become" – rekurencja z nowym stanem
        {get, From} ->
            From ! {value, N},
            loop(N);
        stop -> ok
    after 5000 -> io:format("bezczynny~n"), loop(N)
    end.

% Pid = counter:start(), Pid ! {inc, self()}.
```

#### Akka (Typed, Scala)
```scala
object Counter {
  sealed trait Command
  final case class Increment(replyTo: ActorRef[Int]) extends Command

  def apply(n: Int = 0): Behavior[Command] =
    Behaviors.receiveMessage {
      case Increment(replyTo) =>
        replyTo ! (n + 1)          // tell – asynchronicznie
        Counter(n + 1)             // nowe zachowanie ze zmienionym stanem
    }
}
```
Wzorce: **tell** (wyślij i zapomnij), **ask** (żądanie–odpowiedź przez `Future`), routery, event sourcing aktorów (Akka Persistence), klaster z shardingiem aktorów.

### Problemy
- **Zakleszczenia logiczne** – aktor A czeka (ask z blokowaniem) na B, a B na A,
- **nieograniczone skrzynki** → przepełnienie pamięci przy szybszym wysyłaniu niż przetwarzaniu (potrzeba przeciwciśnienia),
- **utrata komunikatów** w systemie rozproszonym (semantyka at-most-once w Akka Remote) – potwierdzenia i ponowienia po stronie aplikacji,
- trudniejsze **śledzenie przepływu** i debugowanie niż wywołania synchroniczne,
- operacje obejmujące **stan wielu aktorów atomowo** – wymagają protokołów (sagi, 2PC).

---
## Rachunek pi (π-calculus)
Robin **Milner**, Joachim Parrow, David Walker (1992) – rozwinięcie **CCS** (_Calculus of Communicating Systems_, Milner 1980).

> **Rachunek pi** to algebra procesów. Opisuje systemy współbieżne jako **procesy** komunikujące się przez **kanały** (nazwy). Kluczowa cecha to **mobilność**: przez kanały przesyła się **nazwy kanałów**, więc struktura połączeń między procesami **zmienia się w trakcie obliczenia**. Nazwy są jedynym rodzajem danych.

### Składnia
$$P, Q ::= 0 \;\mid\; \bar{x}\langle y \rangle.P \;\mid\; x(z).P \;\mid\; \tau.P \;\mid\; P \mid Q \;\mid\; (\nu x)P \;\mid\; !P \;\mid\; P + Q \;\mid\; [x = y]P$$

| Konstrukcja | Znaczenie |
|---|---|
| $0$ | proces pusty (bezczynny, zakończony) |
| $\bar{x}\langle y \rangle.P$ | **wyślij** nazwę $y$ kanałem $x$, potem zachowuj się jak $P$ (prefiks wyjścia) |
| $x(z).P$ | **odbierz** nazwę kanałem $x$ i **zwiąż** ją z $z$ w $P$ (prefiks wejścia – $z$ zmienna związana) |
| $\tau.P$ | akcja wewnętrzna (niewidoczna), potem $P$ |
| $P \mid Q$ | **kompozycja równoległa** – $P$ i $Q$ działają współbieżnie i mogą się komunikować |
| $(\nu x)P$ | **restrykcja** – utworzenie **nowej, prywatnej** nazwy $x$ o zasięgu $P$ |
| $!P$ | **replikacja** – nieskończenie wiele kopii $P$ równolegle ($!P \equiv P \mid !P$) – odpowiednik rekurencji/serwera |
| $P + Q$ | **wybór niedeterministyczny** (sumy strzeżone prefiksami) |
| $[x = y]P$ | **dopasowanie** – $P$, jeśli nazwy są równe |

**Nazwy wolne** $fn(P)$ i **związane** $bn(P)$ (przez wejście i restrykcję), podstawienie $P\{y/z\}$ z zamianą nazw związanych (α-konwersja) dla uniknięcia przechwycenia.

### Semantyka redukcyjna
**Reguła komunikacji** (jedyna „prawdziwa” reguła obliczeń):
$$\bar{x}\langle y \rangle.P \;\mid\; x(z).Q \;\longrightarrow\; P \;\mid\; Q\{y/z\}$$
Komunikacja jest **synchroniczna** (_rendezvous_ – nadawca i odbiorca spotykają się jednocześnie) i **anonimowa** (odbiorca nie wie, kto wysłał).

Reguły strukturalne:
$$\frac{P \longrightarrow P'}{P \mid Q \longrightarrow P' \mid Q} \qquad \frac{P \longrightarrow P'}{(\nu x)P \longrightarrow (\nu x)P'} \qquad \frac{P \equiv P' \quad P' \longrightarrow Q' \quad Q' \equiv Q}{P \longrightarrow Q}$$
$$\tau.P + M \longrightarrow P$$

**Kongruencja strukturalna** $\equiv$ (procesy „z definicji takie same”):
- $P \mid Q \equiv Q \mid P$, $\;(P \mid Q) \mid R \equiv P \mid (Q \mid R)$, $\;P \mid 0 \equiv P$,
- $!P \equiv P \mid !P$,
- $(\nu x)0 \equiv 0$, $\;(\nu x)(\nu y)P \equiv (\nu y)(\nu x)P$,
- **rozszerzenie zasięgu** (_scope extrusion_): $(\nu x)(P \mid Q) \equiv (\nu x)P \mid Q$, jeśli $x \notin fn(Q)$,
- α-konwersja nazw związanych.

### Mobilność – przykłady
**1. Przekazanie kanału (zmiana topologii)**:
$$\bar{a}\langle c \rangle.0 \;\mid\; a(x).\bar{x}\langle v \rangle.0 \;\mid\; c(y).R$$
$$\longrightarrow\; 0 \;\mid\; \bar{c}\langle v \rangle.0 \;\mid\; c(y).R \;\longrightarrow\; R\{v/y\}$$
Drugi proces początkowo **nie znał** kanału $c$ – otrzymał go przez $a$ i użył do komunikacji z trzecim.

**2. Rozszerzenie zasięgu prywatnej nazwy (bezpieczny kanał)**:
$$(\nu k)(\bar{a}\langle k \rangle.\bar{k}\langle sekret \rangle.0) \;\mid\; a(x).x(s).Q$$
$$\equiv (\nu k)(\bar{a}\langle k \rangle.\bar{k}\langle sekret \rangle.0 \mid a(x).x(s).Q) \;\longrightarrow\; (\nu k)(\bar{k}\langle sekret \rangle.0 \mid k(s).Q\{k/x\}) \;\longrightarrow\; (\nu k)\,Q\{k/x, sekret/s\}$$
Świeża nazwa $k$ jest współdzielona tylko przez dwa procesy – nikt inny nie może podsłuchać kanału $k$. To idea **spi-calculus** (Abadi, Gordon) do analizy protokołów kryptograficznych.

**3. Serwer z replikacją i kanałem odpowiedzi**:
$$!\, req(x, r).\bar{r}\langle f(x) \rangle.0 \;\mid\; (\nu r)(\overline{req}\langle 5, r \rangle.\, r(y).P)$$
Klient tworzy prywatny kanał odpowiedzi $r$ – odpowiednik adresu zwrotnego / wywołania procedury (kodowanie RPC i funkcji).

### Równoważność procesów
- **Bisymulacja** – relacja $\mathcal{R}$ taka, że jeśli $P \mathcal{R} Q$ i $P$ wykonuje akcję $\alpha$ do $P'$, to $Q$ wykonuje $\alpha$ do $Q'$ z $P' \mathcal{R} Q'$ (i symetrycznie). **Silna** (licząc akcje $\tau$) i **słaba** (ignorując $\tau$, $\approx$); w pi: bisymulacje wczesne/późne/otwarte (różne traktowanie nazw).
- Służy do dowodzenia, że **implementacja** zachowuje się jak **specyfikacja**.

### Siła wyrazu i warianty
- Rachunek pi jest **Turing-zupełny**; da się w nim zakodować **rachunek lambda** (Milner – kodowanie wywołania przez wartość/nazwę), struktury danych (listy, liczby), obiekty, stan (komórka pamięci jako proces), wybór i rekurencję.
- **Asynchroniczny rachunek pi** (Honda-Tokoro, Boudol) – wyjście bez kontynuacji $\bar{x}\langle y \rangle$ → bliższy aktorom i komunikacji sieciowej.
- **Wielomianowy (polyadic)** – krotki nazw $\bar{x}\langle y_1, \dots, y_n \rangle$, z typami **sortów**.
- **Typy sesji** (_session types_, Honda) – typowanie protokołów komunikacji (kolejność komunikatów, zgodność), podstawa weryfikacji protokołów i języków (Rust `session-types`, Scribble).
- **Stochastyczny rachunek pi** – modelowanie systemów biologicznych (sieci reakcji).
- **Zastosowania**: semantyka języków (Pict, Occam-pi, JoCaml – join-calculus), modelowanie **procesów biznesowych i usług** (inspiracja dla BPEL, WS-CDL – [[Technologie internetowe w przetwarzaniu rozproszonym/BEPL]]), weryfikacja protokołów bezpieczeństwa (ProVerif – applied pi-calculus), narzędzia: Mobility Workbench, ABC.

### Porównanie z CSP i CCS
| | CSP (Hoare 1978) | CCS (Milner 1980) | π-calculus (1992) |
|---|---|---|---|
| Komunikacja | synchroniczna, kanały nazwane | synchroniczna, kanały | synchroniczna, kanały |
| Dane w kanale | wartości | brak (tylko synchronizacja) | **nazwy kanałów** |
| Topologia | statyczna | statyczna | **dynamiczna (mobilność)** |
| Implementacje | Occam, Go (kanały inspirowane CSP), Clojure core.async | – | Pict, Occam-pi |

---
## Model aktorów a rachunek pi
| Cecha | Model aktorów | Rachunek pi |
|---|---|---|
| Podstawowa jednostka | **aktor** (tożsamość, stan, skrzynka) | **proces** (anonimowy) |
| Adresowanie | adres **aktora** | **kanał** (nazwa), niezależny od procesów |
| Komunikacja | **asynchroniczna**, buforowana w skrzynce | **synchroniczna** (rendezvous); asynchroniczna w wariancie |
| Odbiorcy | dokładnie jeden aktor na adres | dowolnie wielu procesów może słuchać na kanale (niedeterminizm) |
| Stan | jawny, zmieniany przez `become` | niejawny (kodowany strukturą procesów) |
| Mobilność | przesyłanie adresów aktorów | przesyłanie nazw kanałów |
| Tworzenie | `create`/`spawn` aktora | `(ν x)` – nowa nazwa, `!` – replikacja |
| Charakter | model **programowania** i architektury systemów (Erlang, Akka) | **formalizm matematyczny** do rozumowania i weryfikacji |
| Uczciwość / dostarczenie | dostarczenie gwarantowane (w teorii) | brak gwarancji uczciwości wyboru |
| Równoważność | semantyka Aghy, bisymulacja aktorów | bisymulacje, kongruencje |

Oba modele **odrzucają pamięć współdzieloną** na rzecz komunikatów, co eliminuje wyścigi danych i ułatwia rozproszenie. Asynchroniczny rachunek pi jest bliski modelowi aktorów (nazwa kanału ≈ adres skrzynki).

## Zobacz też
- [[16 Asynchroniczna implementacja serwerów usług]] – aktorzy jako abstrakcja serwerów
- [[08 Podejścia do budowy systemów rozproszonych]]
