---
up: 
class: MBP
---
### Notatki z prezentacji "Linearizability" dr hab. inż. Pawła T. Wojciechowskiego

#### 1. Wprowadzenie do linearizowalności

**Linearizability (Herlihy and Wing, 1990)**
- **Definicja:** Obiekt współbieżny jest linearizowalny, jeśli każda operacja wydaje się zachodzić natychmiastowo w pewnym punkcie czasowym między jej wywołaniem a zakończeniem.
- **Punkt linearizacji:** Wewnątrz każdej metody identyfikuje się punkt, w którym wywołanie metody można uznać za skuteczne (np. zwolnienie blokady, operacje load, store, CAS).
- **Kolejność:** Jeśli punkt linearizacji operacji \(m1\) poprzedza punkt linearizacji operacji \(m2\), \(m1\) linearizuje się przed \(m2\).
- **Spójność:** Zachowanie operacji w pojedynczej całkowitej kolejności punktów linearizacji musi być zgodne z sekwencyjną semantyką obiektu.

#### 2. Podstawowe pojęcia i definicje

- **Historia:** Model wykonania systemu współbieżnego, czyli skończona sekwencja zdarzeń wywołania i odpowiedzi metod.
- **Podhistoria:** Podsekwencja zdarzeń historii \(H\).
- **Wywołanie metody:** Para składająca się z wywołania \(inv(m)\) i pasującej odpowiedzi \(res(m)\) w \(H\).
- **Historia sekwencyjna:** Historia, w której każde wywołanie, poza ostatnim, jest bezpośrednio poprzedzone pasującą odpowiedzią.
- **Historia obiektu:** Historia, w której wszystkie zdarzenia są związane z tym samym obiektem.
- **Specyfikacja sekwencyjna:** Zbiór sekwencyjnych historii obiektu zamknięty na prefiksy.
- **Legalna historia sekwencyjna:** Każda podhistoria obiektu \(H|x\) należy do specyfikacji sekwencyjnej obiektu \(x\).

#### 3. Formalna definicja linearizowalności

Historia \(H\) jest linearizowalna, jeśli istnieje jej rozszerzenie \(H'\) i legalna historia sekwencyjna \(S\), taka że:
1. \(complete(H')\) jest równoważne \(S\).
2. Jeśli wywołanie metody \(m0\) poprzedza wywołanie metody \(m1\) w \(H\), to tak samo jest w \(S\).

Obiekt współbieżny jest linearizowalny, jeśli jego historie współbieżne są linearizowalne względem pewnej specyfikacji sekwencyjnej.

#### 4. Przykłady historii i ich analiza

**Przykłady historii:**
- Historia może być sekwencyjnie spójna, ale nie linearizowalna. Linearizowalność wymaga zgodności z rzeczywistym porządkiem czasowym operacji.

#### 5. Właściwości i zalety linearizowalności

- **Kompozycyjność:** Implementacje linearizowalnych obiektów współbieżnych są kompozycyjne. Linearizowalność systemu jako całości zależy tylko od lokalnej linearizowalności jego części.
- **Nieblokująca:** Linearizowalność sama w sobie nie wymusza blokowania wątków z oczekującymi wywołaniami metod totalnych.

#### 6. Ograniczenia linearizowalności

- **Operacje na wielu obiektach:** Linearizowalność operacji na pojedynczych obiektach nie zapewnia linearizowalności operacji na wielu obiektach, które mają być wykonywane atomowo.

#### 7. Transakcje i serializowalność

- **Transakcje:** Możliwość łączenia mniejszych operacji atomowych w większe operacje na wielu obiektach.
- **Serializowalność:** Transakcje są serializowalne, jeśli mają taki sam efekt, jakby były wykonywane pojedynczo w pewnym całkowitym porządku.

#### 8. Porównanie właściwości porządkowania

- **Sequential Consistency (SC)**
- **Linearizability (L)**
- **Serializability (S)**
- **Strict Serializability (SS)**

| Właściwość | SC | L | S | SS |
|------------|----|---|---|----|
| Odpowiada kolejności sekwencyjnej | + | + | + | + |
| Szanuje porządek programu w wątku | + | + | - | + |
| Zgodne z "rzeczywistym" porządkiem czasowym | - | + | - | + |
| Obsługa wielu obiektów atomowo | - | - | + | + |
| Kompozycyjność | - | + | - | - |

#### 9. Właściwości żywotności – blokowanie i nieblokowanie

- **Blokowanie:** Metoda obiektu jest blokująca, jeśli istnieje stan systemu, w którym wątek, który wywołał metodę, nie może zwrócić odpowiedzi, dopóki inny wątek nie wykona jakiejś akcji.
- **Nieblokowanie:** Metoda jest nieblokująca, jeśli nie ma stanu systemu, w którym wywołanie metody nie może zakończyć się i zwrócić odpowiedzi.

### Kluczowe koncepty

1. **Linearizowalność:** Kluczowa właściwość dla poprawności obiektów współbieżnych, zapewniająca, że operacje wydają się zachodzić w określonej kolejności.
2. **Historia i podhistoria:** Podstawowe narzędzia do modelowania wykonania systemów współbieżnych.
3. **Kompozycyjność:** Linearizowalne implementacje mogą być projektowane i weryfikowane niezależnie.
4. **Transakcje i serializowalność:** Ważne dla operacji obejmujących wiele obiektów, które muszą być wykonywane atomowo.

Te notatki powinny pomóc w zrozumieniu i zapamiętaniu kluczowych pojęć związanych z linearizowalnością, co jest istotne na egzamin.

linearizability - operacje "wyglądają" jakby były wykonywane sekwencyjnie,  definiowana w czasie rzeczywistym, określa punkty w których dane operację się wykonują, jeśli operację na siebie nachodzą, ich kolejność może być w dowolnej kolejności

serializabilty - wiele transakcji wykonanych współbieżnie daje taki sam efekt jak by były wykonane sekwencyjnie