---
tags:
  - obrona
up: "[[SWN 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 22
---
# 22. Wsteczne odtwarzanie stanu przetwarzania rozproszonego — pojęcia
---
> Skrót pojęciowy. Algorytmy są tylko nazwane i zlinkowane — opis w [[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane|SWN 01]]–[[SWN 04 Algorytmy odtwarzania - opis szczegółowy|SWN 04]].

## Awarie
**procesu** — zakleszczenie, timeout, błąd ochrony, niespójność → abort, restart
**węzła** — błędy programowe/sprzętowe, zasilanie → stop i restart ze zdefiniowanego stanu
**pamięci masowej** → rekonstrukcja z archiwum
**komunikacyjne** — awaria medium lub urządzeń → naprawa, retransmisja

Typy awarii systemowych w modelu **fail-recovery**:
**przerwa** — restart w **tym samym** stanie sprzed awarii
**amnezja** — restart w stanie **predefiniowanym**
**częściowa amnezja** — część stanu zachowana, reszta predefiniowana

## Odtwarzanie postępowe i wsteczne
**postępowe** (*forward*) — błąd usuwany ze stanu i przetwarzanie postępuje dalej
→ wymaga przewidzenia błędu, projektowane pod konkretny system, **nieimplementowalne jako mechanizm systemowy**
**wsteczne** (*backward*) — cały stan wymieniany na wcześniej zarejestrowany
→ **uniwersalne**, niezależne od rodzaju błędu, implementowalne ogólnie

**Wymagania wstecznego:** stan poprawnie przywrócony **oraz** poprzedzający wystąpienie uszkodzenia.
**Cena:** *narzut* · *nawrót* (brak gwarancji, że awaria się nie powtórzy) · *niepowtarzalność* — interakcje zewnętrzne są nieodwracalne.

## Odtwarzanie węzła
**Wycofywanie operacji** (*operation-based*) — log $\langle OBJ, UNDO, REDO \rangle$; operacje **do** / **undo** / **redo**
 **updating-in-place** — zapis i log jednocześnie → **brak atomowości** (awaria między update a log)
 **write-ahead-log** — update **po** zapisaniu UNDO; REDO **przed** zatwierdzeniem → atomowość

**Przywracanie stanu** (*state-based*) — przechowywany pełen stan
 **shadow pages** — zapis do kopii roboczej, oryginał jako kopia zapasowa
 **twin-page** — dwie kopie, zapisy naprzemiennie
 **checkpointing**

## Punkt kontrolny
**punkt kontrolny** — stan wykonania zachowany w **pamięci trwałej** w celu wznowienia od niego
**lokalny** $cp_i$ — punkt kontrolny procesu $P_i$ utworzony przez kontroler $C_i$
**globalny** $CP = \langle cp_i : \forall P_i \rangle$ — wektor lokalnych
**para punktów kontrolnych** — fragment $CP$ złożony z 2 lokalnych
**CPP** — zbiór wszystkich par · **CCPP** — zbiór par spójnych, $CCPP \subseteq CPP$
**interwał punktu kontrolnego** $I_i^{k}$ — odcinek wykonania $P_i$ zakończony punktem $cp_i^{k}$

## Patologie
**wiadomość osierocona** — $recv(m) \in cp_j \wedge send(m) \notin cp_i$
→ **psuje spójność**; może naruszyć bezpieczeństwo (np. dwa procesy z tym samym żetonem)
**wiadomość utracona** — $send(m) \in cp_i \wedge recv(m) \notin cp_j$
→ **nie psuje spójności**; wymaga mechanizmu odtworzenia znaczących wiadomości
**duplikacja** — deterministyczne ponowienie zdarzenia wysłania po wycofaniu
**niespójność stanu kanałów** — przy niedeterminizmie w miejsce $m_1$ leci $m_2$, a $m_1$ wciąż jest w kanale
**efekt domino** — kolejne wycofania wymuszają wycofania sąsiadów kaskadowo, aż do początku przetwarzania
**ciągły restart** (*recovery livelock*) — każdy restart rodzi nową wiadomość osieroconą
**metoda kolejnych wycofań** — szukanie wcześniejszego $CP$ spełniającego wymogi poprawności

## Spójność i linia odtwarzania
**$CP^\bullet$** (spójny) — $(e \in CP \wedge e' \rightarrow e) \Rightarrow e' \in CP$
**$CP^{!}$** (silnie spójny) — $CP^\bullet$ **+** $send(m) \in CP \Rightarrow recv(m) \in CP$
→ $CP^\bullet$ dopuszcza wiadomości utracone; $CP^{!}$ wymaga **pustych kanałów**

$CP$ jest spójny $\iff \bigwedge_{1 \leqslant i,j \leqslant n} (cp_i, cp_j) \in CCPP$

**GC** — zbiór globalnych punktów kontrolnych · **$GC^\bullet$** — zbiór spójnych, $GC^\bullet \subseteq GC$
**RL** (*recovery line*) — spójny globalny punkt odtwarzania, $RL \in GC^\bullet$
**$RL^{*}$** — najświeższa: $\bigwedge_{RL \in GC^\bullet} RL \subseteq RL^{*}$
**punkt odtwarzania** $rp_i$ — należący do jakiejś RL; może leżeć **pomiędzy** punktami kontrolnymi, osiągany odtworzeniem zdarzeń z logu
**wirtualny punkt odtwarzania** $vcp_i$ — bieżący stan uznany za punkt kontrolny bez zapisu
**zawężenie RL** — linia obejmuje tylko część procesów

**output commit** — protokół gwarantujący, że stan, w którym wykonano operację na świecie zewnętrznym, **nigdy nie będzie wycofany**; dane pobrane z zewnątrz muszą trafić do pamięci trwałej. Wchodzi w skład **każdego** protokołu odtwarzania z interakcjami zewnętrznymi.

## Techniki tworzenia punktów kontrolnych
**skoordynowany** (synchroniczny) — $CP$ zawsze jest $CP^\bullet$
 wady: (1) dodatkowe wiadomości kontrolne · (2) narzut synchronizacji · (3) zbędny koszt przy małej liczbie awarii · (4) koszt ponoszony **nawet gdy awarie nie występują**
**niezależny** (asynchroniczny) — każdy $C_i$ decyduje sam → brak gwarancji $CP^\bullet$, **ryzyko efektu domino**, RL trzeba **odszukać**
**hybrydowy** — asynchroniczny **+** okazjonalna synchronizacja ustalająca RL
**quasi-synchroniczny** — dąży do synchronizacji, nie zawsze ją osiąga
 **podstawowy punkt kontrolny** (*basic*) — wyznaczany niezależnie, co interwał
 **wymuszony punkt kontrolny** (*forced*) — implikowany komunikacją, **przed dostarczeniem** wiadomości; przesuwa RL do przodu

**rodzaje punktów w protokołach dwufazowych:** **ostateczny** (*permanent*, należy do $CP^\bullet$, tylko taki może być punktem odtwarzania) · **wstępny** (*tentative*, zanim stanie się ostateczny)

→ **[[SWN 01 Odtwarzanie i punkty kontrolne skoordynowane#Algorytm Koo-Touega — idea i rodzaje punktów kontrolnych|Koo-Toueg]]** — skoordynowane tworzenie CP tak, by zawsze był to $CP^\bullet$; przetwarzanie dyfuzyjne + 2PC · pseudokod: [[SWN 04 Algorytmy odtwarzania - opis szczegółowy#Algorytm Koo-Toueg|SWN 04]] · [[Systemy Wysokiej Niezawodności/Algorytm Koo-Touega|vault]]
→ **[[SWN 03 Checkpointing hybrydowy i quasi-synchroniczny#Algorytm Manetho|Manetho]]** — checkpointing hybrydowy: izolacja procesu od awarii innych, rollback tylko do ostatniego CP · pseudokod: [[SWN 04 Algorytmy odtwarzania - opis szczegółowy#Algorytm Manetho|SWN 04]] · [[Systemy Wysokiej Niezawodności/Algorytm Manetho|vault]]
→ **[[SWN 03 Checkpointing hybrydowy i quasi-synchroniczny#Checkpointing quasi-synchroniczny — algorytm Manivannana-Singhala|Manivannan-Singhal]]** — każdy CP jest $CP^\bullet$, brak efektu domina, odtwarzanie bez czekania na innych · [[Systemy Wysokiej Niezawodności/Algorytm Manivannana-Singhala|vault]]

## Logowanie komunikatów
Cel: **ograniczyć rollback** i **uniknąć efektu domino** — po wycofaniu odtworzyć zapisane wiadomości aż do stanu bez osieroconych.

**u odbiorcy** — `receive` jest niedeterministyczne; odtworzenie przyspiesza ponowne wykonanie; retransmisja może być zbędna
**u nadawcy** — usprawnia retransmisję, umożliwia przetwarzanie niedeterministyczne; przy rozgłaszaniu jeden zapis obsługuje wszystkie kopie
**pesymistyczne** — zapis do pamięci trwałej **atomowo** z `send`/`receive` (w praktyce: log po `receive`, przed `deliver`) → **duży narzut czasowy**, brak osieroconych
**optymistyczne** — zapis do pamięci **ulotnej**, przepisanie później → brak narzutu, ale przy awarii **część wiadomości ginie** wraz z logiem
**przyczynowe** (*causal*) — niezmiennik: informacja o każdym zdarzeniu przyczynowo poprzedzającym stan procesu jest **w pełni zalogowana albo dostępna lokalnie**
**wybiórcze pesymistyczne** — logowane tylko te wiadomości, które trzeba będzie odtworzyć

**graf poprzedzania przyczynowego** (*antecedence graph*) — DAG zdarzeń niedeterministycznych poprzedzających dany stan; doklejany do wiadomości (*piggybacking*), w praktyce **przyrostowo**
**numer inkarnacji** — odróżnia komunikaty sprzed awarii od tych po cofnięciu; nieaktualne są odrzucane
**wiadomość opóźniona** — $m.inc\_num < inc\_num_i \wedge m.ckpt\_num < RL\_num_i$ → przetworzyć
**wiadomość zduplikowana** — $m.inc\_num < inc\_num_i \wedge m.ckpt\_num \geqslant RL\_num_i$ → odrzucić

## Zależność punktów kontrolnych
**bezpośrednia z-zależność** — $cp_j^{\,l}$ jest bezpośrednio z-zależny od $cp_i^{k}$ gdy
$i = j \wedge l = k+1$ **lub** $i \neq j \wedge \exists_m\,(send(m) \in I_i^{k} \wedge recv(m) \in I_j^{\,l-1})$
**z-dependency** — domknięcie przechodnie powyższej
**Tw. 1** — z-zależność dwóch punktów kontrolnych czyni z nich **parę niespójną**
**graf punktów kontrolnych** — węzły = punkty (interwały), krawędzie = bezpośrednia z-zależność
**rozszerzony graf** — zawiera wirtualne punkty kontrolne
**krawędź rollback** — reprezentuje wiadomość **jeszcze nie zapisaną w logu**; węzeł z wchodzącą krawędzią rollback i wszystkie osiągalne z niego są **wykluczane**
**root set** — startowo ostatnie punkty każdego procesu; po iteracyjnym zastępowaniu oznaczonych staje się RL
**wiadomość non-state** — nigdy nie stanie się częścią żadnego stanu kanału → **nie trzeba jej logować**
**Tw. 2** — jeśli istnieje ścieżka z $cp_i^{k}$ do $cp_j^{\,l}$, to wszystkie wiadomości z $I_j^{\,l-1}$ odebrane w $I_i^{k}$ są *non-state*

**Wyznaczanie RL przy punktach niezależnych:**
→ **[[SWN 02 Punkty kontrolne niezależne i logowanie komunikatów#Algorytm Juanga-Venkatesana|Juang-Venkatesan]]** — wyznacza RL z dokładnością do pojedynczych zdarzeń, porównując liczniki wysłanych i odebranych wiadomości · [[Systemy Wysokiej Niezawodności/Algorytm Juanga-Venkatesana|vault]]
→ **[[SWN 02 Punkty kontrolne niezależne i logowanie komunikatów#Algorytm Wanga-Fuchsa|Wang-Fuchs]]** — wyznacza RL z grafu z-zależności przy logowaniu optymistycznym; obsługuje krawędzie rollback · pseudokod: [[SWN 04 Algorytmy odtwarzania - opis szczegółowy#Algorytm Wanga-Fuchsa|SWN 04]] · [[Systemy Wysokiej Niezawodności/Algorytm Wanga-Fuchsa|vault]]

## Odśmiecanie punktów kontrolnych
**discardable** — punkt, który **nigdy** nie będzie należał do żadnej przyszłej RL
**obsolete** — poprzedzający **najgorszą możliwą** RL; każdy *obsolete* jest *discardable*, ale **niektóre non-obsolete też są** *discardable*
**całkowity efekt domino** — RL tworzą wyłącznie początkowe punkty kontrolne → duża liczba punktów *non-obsolete* utrzymywana bez potrzeby
**nadgraf $\hat{G}$** — graf $G$ rozszerzony o **wszystkie** wirtualne punkty kontrolne
**Tw. 3** — punkt w $G$ jest *non-discardable* $\iff$ należy do **sumy linii odtwarzania** wszystkich $\hat{G}\text{-}vcp_i$, $1 \leqslant i \leqslant N$
→ intuicyjnie: należy do którejkolwiek z $N$ linii powstałych przy awarii któregokolwiek procesu
