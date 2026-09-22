---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 1
---
# 1. Komunikacja grupowa
---
> **Komunikacja grupowa** to przesyłanie komunikatu do **grupy procesów** traktowanej jako jeden logiczny odbiorca. Nadawca nie musi znać liczby, położenia ani tożsamości członków grupy. Warstwa komunikacji grupowej odpowiada za niezawodność i **uporządkowanie dostarczania** komunikatów.

## Definicje podstawowe
- **Grupa procesów** – zbiór procesów, do którego komunikat adresowany jest jednym identyfikatorem.
- **Grupa zamknięta** – komunikaty do grupy mogą wysyłać tylko jej członkowie (np. repliki). **Grupa otwarta** – dowolny proces spoza grupy może wysłać komunikat do grupy (np. klienci serwisu replikowanego).
- **Grupa płaska** (wszyscy równorzędni, brak SPoF, kosztowne uzgadnianie) vs **hierarchiczna** (koordynator, prostsze decyzje, SPoF).
- **Grupa statyczna** (stały skład) vs **dynamiczna** (procesy dołączają, odchodzą, ulegają awariom).
- **Usługa członkostwa** (_group membership service_) – utrzymuje aktualny skład grupy, wykrywa awarie.
- **Widok** (_view_) – lista członków grupy w danym momencie; zmiana składu = **zmiana widoku** (_view change_).
- **Wysłanie / odbiór vs dostarczenie** – komunikat **odebrany** (_receive_) przez warstwę komunikacyjną może być buforowany; **dostarczony** (_deliver_) to przekazany aplikacji. Porządek dostarczania może różnić się od porządku odbioru.
- Operacje: **broadcast(m)** – rozgłoszenie do wszystkich, **multicast(g, m)** – do grupy $g$.

## Zegary logiczne (podstawa uporządkowania)
**Relacja poprzedzania przyczynowego** $a \rightarrow b$ (Lamport): $a$ poprzedza $b$ w tym samym procesie lub $a$ to wysłanie, a $b$ odbiór tego samego komunikatu (plus domknięcie tranzytywne). Zdarzenia nieporównywalne są **współbieżne** ($a \parallel b$).

### Zegar skalarny Lamporta
- przed każdym zdarzeniem lokalnym i wysłaniem: $C_i := C_i + 1$, komunikat niesie znacznik $ts = C_i$
- przy odbiorze: $C_i := \max(C_i, ts) + 1$
- własność: $a \rightarrow b \implies C(a) < C(b)$ (implikacja **tylko w jedną stronę**)
- porządek totalny: $(C, id)$ – remisy rozstrzygane identyfikatorem procesu

### Zegar wektorowy
- $V_i[i] := V_i[i] + 1$ przy każdym zdarzeniu, komunikat niesie cały wektor
- przy odbiorze: $V_i[k] := \max(V_i[k], V[k])$ dla każdego $k$, potem $V_i[i]{+}{+}$
- własność: $a \rightarrow b \iff V(a) < V(b)$ – **wykrywa współbieżność**

## Rozgłaszanie niezawodne (_Reliable Broadcast_)
Własności (dla procesów poprawnych):
- **Ważność** (_validity_) – jeśli poprawny proces rozgłasza $m$, to (sam) w końcu dostarczy $m$.
- **Zgodność** (_agreement_) – jeśli jakiś poprawny proces dostarczył $m$, to wszystkie poprawne dostarczą $m$.
- **Integralność** (_integrity_) – każdy komunikat dostarczony co najwyżej raz i tylko jeśli został wcześniej rozgłoszony.

**Zgodne (jednolite) rozgłaszanie niezawodne** (_uniform reliable broadcast_) – zgodność dotyczy **wszystkich** procesów: jeśli dowolny proces (także taki, który później ulegnie awarii) dostarczył $m$, to wszystkie poprawne dostarczą $m$.

### Algorytm (_eager / flooding_)
1. Nadawca wysyła $m$ do wszystkich (kanały niezawodne punkt-punkt).
2. Proces, który **pierwszy raz** odbiera $m$: przesyła $m$ do wszystkich pozostałych, a następnie dostarcza $m$.
3. Duplikaty (rozpoznawane po identyfikatorze $(nadawca, nr)$) są odrzucane.

Koszt $O(N^2)$ komunikatów na rozgłoszenie. Retransmisja przed dostarczeniem gwarantuje zgodność mimo awarii nadawcy w trakcie wysyłania. Wersja **jednolita**: dostarczenie dopiero po potwierdzeniu, że $m$ otrzymała większość (lub wszystkie niepodejrzewane procesy).

## Porządki dostarczania
| Skrót | Nazwa | Warunek |
|---|---|---|
| **RFB** | niezawodne rozgłaszanie **FIFO** | jeśli proces rozgłasza $m$ przed $m'$, to każdy proces dostarczający $m'$ dostarczył wcześniej $m$ (porządek tylko względem **tego samego nadawcy**) |
| **RCB** | niezawodne rozgłaszanie **przyczynowe** | jeśli $m \rightarrow m'$ (rozgłoszenie $m$ poprzedza przyczynowo rozgłoszenie $m'$), to każdy proces dostarcza $m$ przed $m'$ |
| **RTB / ABCAST** | niezawodne rozgłaszanie **totalne (atomowe)** | wszystkie procesy dostarczają te same komunikaty **w tej samej kolejności** |

Zależności:
- RCB ⇒ RFB (porządek przyczynowy zawiera FIFO)
- RTB **nie implikuje** FIFO ani przyczynowości – stąd warianty **FIFO-total** i **causal-total**
- rozgłaszanie totalne jest **równoważne konsensusowi** (każde można zbudować z drugiego)

> Zadanie domowe z przykładem historii spełniającej RFB, ale nie RCB: [[broadcast_zad_dom.excalidraw]]. Kontrprzykład typowy: $P_1$ rozgłasza $m_1$, $P_2$ po dostarczeniu $m_1$ rozgłasza $m_2$ (odpowiedź), a $P_3$ dostarcza $m_2$ przed $m_1$ – FIFO spełnione (różni nadawcy), przyczynowość naruszona.

## Algorytmy uporządkowania
### FIFO – numery sekwencyjne
- Nadawca $P_i$ numeruje swoje rozgłoszenia $1, 2, 3, \dots$
- Odbiorca trzyma $next[i]$ (numer oczekiwany od $P_i$) i bufor. Dostarcza $m$ o numerze $s$ od $P_i$ gdy $s = next[i]$, wtedy $next[i]{+}{+}$ i sprawdza bufor.

### Przyczynowe – zegary wektorowe (Birman-Schiper-Stephenson, CBCAST)
- $V_j[k]$ = liczba komunikatów od $P_k$ dostarczonych przez $P_j$.
- $P_i$ przed rozgłoszeniem: $V_i[i]{+}{+}$, dołącza $VT = V_i$.
- $P_j$ dostarcza $m$ od $P_i$ gdy:
  - $VT[i] = V_j[i] + 1$ (to następny komunikat od $P_i$),
  - $VT[k] \leq V_j[k]$ dla $k \neq i$ (dostarczył już wszystko, co widział nadawca).
- Po dostarczeniu: $V_j[i] := VT[i]$; komunikaty niespełniające warunku czekają w buforze.

### Totalne – sekwencer
- **Stały sekwencer**: każdy komunikat trafia do procesu-sekwencera, który nadaje globalny numer i rozgłasza; odbiorcy dostarczają w kolejności numerów. Prosty, ale SPoF i wąskie gardło.
- **Ruchomy sekwencer (token)**: numer nadaje proces aktualnie posiadający token (np. Totem, protokół pierścieniowy).

### Totalne – ISIS (Skeen, uzgadnianie priorytetów)
1. Nadawca rozsyła $m$.
2. Każdy odbiorca $P_j$ proponuje priorytet $p_j = \max(A_j, P_j) + 1$ ($A_j$ – największy uzgodniony, $P_j$ – największy zaproponowany), wstawia $m$ do kolejki jako **niedostarczalny** i odsyła propozycję.
3. Nadawca wybiera $a = \max_j p_j$ (remis – po id) i rozsyła jako **ostateczny**.
4. Odbiorca ustawia priorytet $m$ na $a$, oznacza jako **dostarczalny**, przesortowuje kolejkę; dostarcza komunikaty z **czoła kolejki**, dopóki są dostarczalne.

Koszt $3N$ komunikatów, brak SPoF.

### Totalne – przez konsensus (Chandra-Toueg)
Procesy w rundach uzgadniają (konsensus) **zbiór** komunikatów do dostarczenia; zbiór jest dostarczany w deterministycznej kolejności (np. po id). Działa z detektorem awarii ◇S – zob. [[23 Rozproszone uzgadnianie w środowisku zawodnym]].

## Synchronizacja widoków (_virtual synchrony_)
- Zmiany składu grupy są uporządkowane względem komunikatów: każdy komunikat jest dostarczony **w tym samym widoku** przez wszystkie procesy, które przechodzą do kolejnego widoku.
- Przy zmianie widoku wykonywany jest **flush**: procesy przesyłają sobie komunikaty niestabilne (niepotwierdzone przez wszystkich), a dopiero potem instalują nowy widok.
- Implementacje: ISIS, Horus, JGroups, Spread.

## Rozgłaszanie epidemiczne
Alternatywa probabilistyczna (dobrze skalowalna, bez gwarancji deterministycznych) – [[Algorytmy Rozproszone/Gossiping]].

## Zobacz też
- [[Systemy Wysokiej Niezawodności/Replikacja Procesu]] – replikacja aktywna wymaga rozgłaszania totalnego
- [[02 Danocentryczne modele spójności]]
