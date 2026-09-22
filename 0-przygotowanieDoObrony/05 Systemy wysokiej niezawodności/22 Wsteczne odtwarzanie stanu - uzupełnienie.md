---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 22
---
# 22. Wsteczne odtwarzanie stanu przetwarzania rozproszonego – uzupełnienie
---
> Uzupełnienie istniejących notatek o algorytmach ([[Systemy Wysokiej Niezawodności/Algorytm Koo-Touega|Koo-Toueg]], [[Systemy Wysokiej Niezawodności/Algorytm Juanga-Venkatesana|Juang-Venkatesan]], [[Systemy Wysokiej Niezawodności/Algorytm Wanga-Fuchsa|Wang-Fuchs]], [[Systemy Wysokiej Niezawodności/Algorytm Manivannana-Singhala|Manivannan-Singhal]], [[Systemy Wysokiej Niezawodności/Algorytm Manetho|Manetho]]) o **definicje podstawowe** i **systematykę metod**.

## Wsteczne odtwarzanie w systemie rozproszonym – problem
W systemie scentralizowanym wystarczy przywrócić stan z punktu kontrolnego ([[Systemy Wysokiej Niezawodności/Odtwarzanie Wsteczne]]). W systemie rozproszonym procesy **wymieniają komunikaty**, więc wycofanie jednego procesu może unieważnić zdarzenia w innych. Trzeba znaleźć zbiór lokalnych stanów tworzących **spójny stan globalny**.

## Definicje
- **Stan globalny** $S = \langle s_1, \dots, s_N \rangle$ – wektor stanów lokalnych wszystkich procesów (plus stany kanałów).
- **Przekrój** (_cut_) – zbiór zdarzeń zawierający prefiks historii każdego procesu; granica przekroju odpowiada stanowi globalnemu.
- **Spójny stan globalny / przekrój spójny** – jeśli zawiera zdarzenie **odbioru** komunikatu, to zawiera też jego **wysłanie**. Formalnie: $e \in C \wedge e' \rightarrow e \implies e' \in C$ (domknięty ze względu na poprzedzanie przyczynowe).
- **Komunikat osierocony** (_orphan message_) – w stanie globalnym jego **odbiór jest zarejestrowany**, a **wysłanie nie** (nadawca wycofał się przed wysłanie). Obecność sierot = **niespójność**.
- **Komunikat zagubiony / w tranzycie** (_lost / in-transit message_) – **wysłanie zarejestrowane**, a **odbiór nie**. Stan jest spójny, ale komunikat trzeba **odtworzyć** (logowanie komunikatów w kanale) przy zawodnych kanałach lub gdy wymagane dostarczenie.
- **Punkt kontrolny** (_checkpoint_) $cp_i$ – stan procesu zapisany w pamięci trwałej ([[Systemy Wysokiej Niezawodności/Checkpoint]]). Globalny punkt kontrolny $CP = \langle cp_1, \dots, cp_N \rangle$.
- **Spójny globalny punkt kontrolny** $CP^\bullet$ – taki $CP$, który jest spójnym stanem globalnym (brak sierot między punktami).
- **Linia odtwarzania** (_recovery line_, $RL$) – **najpóźniejszy** spójny globalny punkt kontrolny, do którego można wycofać system.
- **Efekt domina** – przy niezależnych punktach kontrolnych kaskada wycofań: wycofanie $P_i$ tworzy sieroty u $P_j$, $P_j$ wycofuje się dalej, co tworzy sieroty u kolejnych… W skrajnym przypadku system wraca do **stanu początkowego** i cała praca jest tracona.
- **Punkt kontrolny bezużyteczny** (_useless_) – nie może należeć do żadnego spójnego $CP$ (występuje w cyklu **Z-ścieżek**); Z-ścieżki uogólniają ścieżki przyczynowe i pozwalają analizować zależności między punktami kontrolnymi.
- **Przetwarzanie fragmentarycznie deterministyczne** (_piecewise deterministic, PWD_) – przetwarzanie składa się z interwałów deterministycznych, każdy rozpoczyna się zdarzeniem niedeterministycznym (np. odbiorem komunikatu). Zarejestrowanie tych zdarzeń (determinantów) pozwala **odtworzyć** wykonanie (_replay_). Podstawa logowania komunikatów.
- **Output commit** – interakcji ze **światem zewnętrznym** (wydruk, wydanie gotówki) nie da się wycofać. Przed wysłaniem wyniku na zewnątrz system musi zapewnić, że stan, który go wygenerował, **nie zostanie wycofany** (zapisanie punktu kontrolnego / zalogowanie determinantów) – koszt i opóźnienie.
- **Garbage collection** – usuwanie punktów kontrolnych i logów starszych niż linia odtwarzania.

## Systematyka metod
```
Odtwarzanie wsteczne
├── oparte na punktach kontrolnych (checkpoint-based)
│   ├── nieskoordynowane / niezależne (independent) ── Juang-Venkatesan, Wang-Fuchs
│   ├── skoordynowane / synchroniczne (coordinated) ── Koo-Toueg, Chandy-Lamport
│   └── wymuszane komunikacją / quasi-synchroniczne (CIC) ── Manivannan-Singhal
└── oparte na logowaniu komunikatów (log-based) + punkty kontrolne
    ├── pesymistyczne
    ├── optymistyczne
    └── przyczynowe (causal) ── Manetho
```

### Punkty kontrolne – porównanie
| Cecha | Niezależne | Skoordynowane | Wymuszane komunikacją (CIC) |
|---|---|---|---|
| Tworzenie | każdy proces sam, kiedy chce | wszystkie razem (protokół, np. 2-fazowy) | lokalne + **wymuszone** na podstawie informacji w komunikatach |
| Narzut przy pracy | minimalny | koordynacja, blokowanie (lub znaczniki) | informacja w komunikatach, dodatkowe punkty |
| Efekt domina | **możliwy** | brak | **brak** (zapobiega Z-cyklom) |
| Punkty bezużyteczne | możliwe | brak | brak |
| Pamięć trwała | wiele punktów na proces + zależności | 1–2 punkty na proces | kilka |
| Odtwarzanie | wyznaczanie RL (graf zależności / rollback-dependency graph) | proste – ostatni $CP^\bullet$ | ostatnie punkty z danym numerem |
| Output commit | kosztowny (trzeba znaleźć RL) | trzeba wykonać globalny punkt | średni |

**Koordynacja bez blokowania** – algorytm migawki **Chandy-Lamport** (kanały FIFO): inicjator zapisuje stan i wysyła **znacznik** (_marker_) wszystkimi kanałami. Proces po pierwszym znaczniku zapisuje stan i rozsyła znaczniki, a następnie rejestruje komunikaty przychodzące każdym kanałem aż do otrzymania nim znacznika (stan kanału). Wynik to spójny stan globalny.

**Obliczenia dyfuzyjne** (Dijkstra-Scholten) – wykorzystywane w Koo-Toueg: inicjator rozsyła żądanie, a każdy proces, który otrzyma je pierwszy raz, przekazuje je dalej swoim sąsiadom (tworząc drzewo rozpinające) i odpowiada rodzicowi dopiero po otrzymaniu odpowiedzi od swoich dzieci. Inicjator wie, że obliczenie się zakończyło, gdy dostanie odpowiedzi od wszystkich.

### Logowanie komunikatów
Rejestruje się **determinanty** zdarzeń niedeterministycznych (identyfikator komunikatu, nadawca, numer kolejny odbioru, ewentualnie treść). Po awarii proces odtwarza się z punktu kontrolnego i **ponownie przetwarza** zalogowane komunikaty w tej samej kolejności (PWD). Cel: wycofać **tylko proces, który uległ awarii**.

| Cecha | Pesymistyczne | Optymistyczne | Przyczynowe |
|---|---|---|---|
| Zasada | determinant zapisany w pamięci trwałej **przed** dostarczeniem komunikatu do aplikacji (synchronicznie) | zapis **asynchroniczny** (bufor ulotny, okresowo na dysk) | determinanty przechowywane w pamięci ulotnej **wielu** procesów, **dołączane do komunikatów** (piggyback) |
| Narzut przy pracy | wysoki (zapis przed każdym dostarczeniem) | niski | średni (większe komunikaty) |
| Sieroty | **nigdy** | możliwe → kaskadowe wycofania innych procesów | **nigdy** (dopóki nie ulegnie awarii więcej procesów niż $f$) |
| Wycofywane procesy | tylko ten po awarii | ten po awarii i zależne od utraconych stanów | tylko ten po awarii |
| Output commit | natychmiast | wymaga analizy zależności | wymaga zapisania determinantów |
| Punkty kontrolne | tylko ostatni | wiele | tylko ostatni |
| Przykład | – | [[Systemy Wysokiej Niezawodności/Algorytm Wanga-Fuchsa\|Wang-Fuchs]], [[Systemy Wysokiej Niezawodności/Algorytm Juanga-Venkatesana\|Juang-Venkatesan]] | [[Systemy Wysokiej Niezawodności/Algorytm Manetho\|Manetho]] (graf poprzedzania przyczynowego) |

Logowanie może być po stronie **nadawcy** (_sender-based_ – nadawca pamięta treść i numer odbioru, odbiorca potwierdza) lub **odbiorcy** (_receiver-based_).

### Przebieg odtwarzania (ogólnie)
1. Wykrycie awarii procesu $P_f$, restart na zapasowym węźle.
2. Przywrócenie stanu z ostatniego punktu kontrolnego.
3. Wyznaczenie linii odtwarzania (algorytm zależny od metody) i poinformowanie innych procesów, które muszą się wycofać.
4. Odtworzenie komunikatów w tranzycie / zalogowanych (replay).
5. Wznowienie przetwarzania; usunięcie zbędnych punktów kontrolnych.

## Algorytm Manivannana-Singhala – pełny algorytm odtwarzania
Uzupełnienie `#TODO` z [[Systemy Wysokiej Niezawodności/Algorytm Manivannana-Singhala]]:
1. $P_f$ po awarii odtwarza się z ostatniego punktu $cp_f$ o numerze $x$; kontroler rozsyła $Rollback(x)$ (numer $rp\_num = x$), zapamiętuje **komunikaty odebrane po $cp_f$ z logu pesymistycznego** i odtwarza je.
2. Każdy $P_j$ po odebraniu $Rollback(x)$:
   - $ckpt\_num_j \geq x$ → cofa się do **najwcześniejszego** swojego punktu z numerem $\geq x$ (usuwając późniejsze),
   - $ckpt\_num_j < x$ → tworzy punkt kontrolny z numerem $x$ (wirtualny – bez faktycznego zapisu, bo stan się nie zmienił), ustawia $ckpt\_num_j := x$, kontynuuje.
3. Komunikaty odebrane przez $P_j$ z numerem interwału $\geq x$ po punkcie odtwarzania są odrzucane / wysyłane ponownie przez nadawców z ich logów.
4. Nowe odtwarzanie inicjowane jest z **numerem wcielenia** (_incarnation_), aby odróżnić komunikaty sprzed awarii i ignorować przestarzałe żądania $Rollback$.
5. Garbage collection: po wycofaniu usuwane są punkty kontrolne sprzed linii odtwarzania.

## Zobacz też
- [[Odtwarzanie]], [[Awarie]], [[Systemy Wysokiej Niezawodności/Odtwarzanie Wsteczne Węzła]]
- [[24 Niezawodne zatwierdzanie transakcji rozproszonych]]
