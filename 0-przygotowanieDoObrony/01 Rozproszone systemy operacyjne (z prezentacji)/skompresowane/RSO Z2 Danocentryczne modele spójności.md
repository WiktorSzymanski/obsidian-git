---
tags:
  - obrona
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 2
---
# 2. Danocentryczne modele spójności
---
> Komplet pojęć zagadnienia. Protokoły realizujące modele są nazwane i opatrzone informacją, co robią, ale ich pseudokod, przykłady historii i zadania pozostają w [[RSO 02 Danocentryczne modele spójności|RSO 02]].

## Zwielokrotnianie i jego cena

**Zwielokrotnianie** polega na utrzymywaniu wielu kopii danych (obiektów) na **niezależnych serwerach**. Robi się to dla dwóch celów: **niezawodności** — odporności na awarie i zwiększenia dostępności — oraz **efektywności**, która rozpada się na współbieżny dostęp do wielu serwerów (równoważenie obciążenia, **skalowalność liczbowa**) i korzystanie z serwerów bliższych (**skalowalność geograficzna**, mniejsze opóźnienia).

Ceną jest **spójność**, i to ona jest właściwą treścią zagadnienia. Zwielokrotnianie a skalowalność to kompromis: skracamy czas dostępu, ale płacimy za utrzymywanie w stanie spójnym także kopii **nieużywanych**; spójność odwzorowująca system scentralizowany wymagałaby transakcyjnej aktualizacji wszystkich kopii i globalnej synchronizacji, więc realnym wyjściem jest **osłabienie modelu spójności** — dobrane do charakteru aplikacji i danych. Cała reszta zagadnienia to katalog takich osłabień.

## Konteksty: DSM i koncepcje dostępu

**DSM** (*Distributed Shared Memory*) to wspólna wirtualna przestrzeń adresowa dostępna dla wszystkich węzłów systemu rozproszonego. Daje wygodny paradygmat programowania równoległego, skalowalność i łatwość rozbudowy, dostęp do pamięci fizycznej wszystkich węzłów oraz środowisko uruchomieniowe dla programów pisanych dla maszyn wieloprocesorowych. Mechanicznie działa jak pamięć wirtualna, z tą różnicą, że **brakująca strona sprowadzana jest z innego węzła sieci**, a nie z urządzenia wymiany.

Dostęp do danych można zorganizować na trzy sposoby. **Dostęp zdalny** — zawsze przez sieć — jest koncepcyjnie i implementacyjnie najprostszy, ale płaci opóźnieniami. **Relokacja** fizycznie przenosi obiekt bliżej, co skraca czas dostępu kosztem przenoszenia i opłaca się przy wielokrotnych, zgrupowanych odwołaniach. **Zwielokrotnianie** tworzy kopie w węzłach lokalnych, skracając czas dostępu za cenę problemu spójności.

Relokację i zwielokrotnianie opisuje ten sam zestaw problemów, ale rozstrzygniętych inaczej. **Problem lokalizacji** przy relokacji oznacza, że adres obiektu zmienia się w czasie, a przy zwielokrotnianiu — że trzeba tworzyć nowe i usuwać stare repliki. **Problem rozmiaru i struktury** przemieszczanej jednostki jest wspólny: małe obiekty dają duży poziom współdzielenia, duże — mały narzut administracyjny. **Problem migotania** (*trashing*, efekt ping-pong), czyli naprzemiennych odwołań kilku procesów, dotyczy **wyłącznie relokacji** — przy zwielokrotnianiu znika, bo każdy ubiegający się węzeł dostaje własną kopię. W zamian pojawia się **problem spójności replik**, którego waga zależy od **stosunku liczby zapisów do odczytów**.

Jednostką zwielokrotniania może być **strona**, **pojedyncza zmienna** albo **obiekt**. Strona łączy fizycznie kilka odrębnych obiektów logicznych w jedną całość, co rodzi **fałszywe współdzielenie** — dwa procesy sięgające po różne obiekty na tej samej stronie wymuszają jej przesyłanie tam i z powrotem. Pojedyncza zmienna daje duży koszt jednostkowy relokacji i utrzymywania spójności. Obiekt, jako hermetyczna struktura dostępna wyłącznie przez zdefiniowane metody, pozwala **optymalizować strategię spójności**, bo sposób dostępu jest z góry znany.

## Protokół koherencji

**Protokół koherencji (spójności)** to algorytm rozproszony realizujący określony model spójności. Model mówi **co** system gwarantuje, protokół — **jak** to zapewnia.

**Protokół unieważniania** (*invalidation protocol*) rozsyła małe komunikaty i unieważnia repliki jednokrotnie. **Protokół aktualizacji** (*update protocol*) przesyła nowe wartości do niespójnych replik, więc jego komunikaty są większe. Wybór między nimi to znów stosunek zapisów do odczytów.

## Model spójności

**Model spójności** określa gwarancje dotyczące spójności replik, dawane aplikacji przez system. Trzy pytania, które trzeba przy nim rozstrzygnąć, to: jak model zdefiniować, jak określić gwarancje dla aplikacji i **kiedy oraz w jaki sposób je egzekwować**.

Punktem wyjścia jest **spójność ścisła** (*strict consistency*): każdy odczyt zmiennej $x$ zwraca wartość zapisaną przez **ostatnią** operację zapisu. W systemie rozproszonym jest ona nieosiągalna z dwóch powodów — słowo „ostatni" jest **niejednoznaczne** przy braku globalnego zegara, a koszt realizacji byłby bardzo duży. Wszystkie dalsze modele są odpowiedzią na to, jak sensownie osłabić spójność ścisłą.

### Klasyfikacja

Modele dzielą się najpierw ze względu na to, **czyj punkt widzenia** opisują. **Modele danocentryczne** (nastawione na dane) dzielą się dalej na modele przy **dostępie ogólnym**, gdzie dane uspójnia się przy każdej modyfikacji, i przy **dostępie synchronizowanym**, gdzie uspójnianie zachodzi tylko podczas jawnych operacji synchronizujących. **Modele nastawione na klienta** uwzględniają mobilność klienta — to osobne zagadnienie: [[RSO Z3 Modele spójności zorientowane na klienta|zagadnienie 3]].

Przy dostępie ogólnym: **spójność atomowa** (*atomic consistency*), zwana też **liniowością** (*linearizability*), **sekwencyjna** (*sequential*), **przyczynowa** (*causal*), **PRAM** (*pipelined RAM*), **podręczna** (*cache consistency*), zwana **koherencją**, oraz **procesorowa** (*processor consistency*). Przy dostępie synchronizowanym: **słaba** (*weak*), **zwalniania** (*release*), **wejścia** (*entry*) i **zakresu** (*scope*) — te prezentacje jedynie wymieniają.

## Formalizm

System DSM to zbiór **sekwencyjnych** procesów $P = \{p_1, \ldots, p_n\}$ i zbiór **współdzielonych zmiennych** $X = \{x_1, x_2, \ldots\}$, przy czym **każdy proces ma własną replikę całego zbioru $X$**. Proces $p_i$ wykonuje na zmiennej $x$ operacje **zapisu** $w_i(x)v$ i **odczytu** $r_i(x)v$, a każda operacja przebiega w **dwóch fazach**: **żądania** (*operation issue*) i **wykonania** (*operation execution*). Rozdzielenie tych faz jest tym, co w ogóle umożliwia rozbieżność porządków.

Oznaczenia: $O$ to zbiór wszystkich operacji, $O_i$ — operacji procesu $p_i$, $OW$ — wszystkich zapisów, $O\vert x$ — operacji na zmiennej $x$. Relacja $\rightarrow_i$ to **lokalny porządek** operacji procesu $p_i$, $\rightarrow$ to **porządek przyczynowy**, a $\mapsto_i$ to **uszeregowanie**, w jakim operacje są postrzegane przez proces $p_i$.

**Historia lokalna** procesu to zbiór **liniowo** uporządkowany $h_i = (O_i, \rightarrow_i)$; **historia globalna** to zbiór **częściowo** uporządkowany $h = (O, \rightarrow)$. **Obraz historii w procesie $p_i$** to zbiór liniowo uporządkowany $hv_i = (O_i \cup OW, \mapsto_i)$ — proces widzi **własne operacje oraz wszystkie zapisy w systemie**. **Obraz historii** to kolekcja $hv = \langle hv_1, \ldots, hv_n \rangle$.

Uszeregowanie $\mapsto_i$ jest **legalne** wtedy, gdy każdy odczyt zwraca wartość pewnego zapisu, a **między tym zapisem a odczytem nie ma w uszeregowaniu innej operacji na tej samej zmiennej o innej wartości**:
$$\forall_{\substack{w(x)v \in OW \\ r(x)v \in O_i}} \left( w(x)v \mapsto_i r(x)v \wedge \nexists_{o(x)u \in O_i \cup OW} [\, u \neq v \wedge w(x)v \mapsto_i o(x)u \mapsto_i r(x)v \,] \right)$$
Dla uproszczenia zakłada się, że **każdy zapis danej zmiennej zapisuje unikalną wartość**, dzięki czemu zapis można identyfikować przez zapisaną wartość.

**Legalność jest warunkiem wspólnym wszystkich modeli** — każdy model to legalność plus dodatkowe warunki nałożone na $\mapsto_i$. Dlatego modele różnią się wyłącznie tym, **ile porządku muszą zachować uszeregowania**.

## Modele przy dostępie ogólnym

Wszystkie warunki niżej dotyczą obrazu $hv$ historii $h$ i zakładają legalność każdego $\mapsto_i$.

**Spójność atomowa (liniowość)** wymaga dwóch rzeczy: zachowania porządku **czasu rzeczywistego** oraz **wspólnego porządku zapisów**.
$$\forall_{o_1, o_2 \in O_i \cup OW} \left( \left( \exists_{j}\ o_1 \rightarrow_{RT} o_2 \right) \Rightarrow o_1 \mapsto_i o_2 \right)$$
$$\forall_{w_1, w_2 \in OW} \left( \forall_{i} w_1 \mapsto_i w_2 \ \vee\ \forall_{i} w_2 \mapsto_i w_1 \right)$$
gdzie $o_1 \rightarrow_{RT} o_2$ znaczy, że $o_1$ **kończy się w czasie rzeczywistym, zanim $o_2$ się zaczyna**.

**Spójność sekwencyjna** ma identyczny warunek drugi, a w pierwszym zastępuje porządek czasu rzeczywistego **porządkiem lokalnym procesów** $\rightarrow_j$:
$$\forall_{o_1, o_2 \in O_i \cup OW} \left( \left( \exists_{j}\ o_1 \rightarrow_j o_2 \right) \Rightarrow o_1 \mapsto_i o_2 \right)$$
To jedyna różnica między atomową a sekwencyjną i najczęstsze pytanie o te dwa modele.

**Spójność przyczynowa** ma **jeden** warunek — zachowanie porządku przyczynowego — i nie wymaga, by wszystkie procesy widziały zapisy w tej samej kolejności:
$$\forall_{o_1, o_2 \in O_i \cup OW} \left( o_1 \rightarrow o_2 \Rightarrow o_1 \mapsto_i o_2 \right)$$

**Spójność PRAM** ma **jeden** warunek — zachowanie porządku lokalnego, czyli dokładnie pierwszy warunek spójności sekwencyjnej **bez** wspólnego porządku zapisów. Intuicyjnie: każdy proces widzi zapisy każdego innego procesu w kolejności ich wykonania, ale zapisy różnych procesów może przeplatać po swojemu.

**Spójność podręczna (koherencja)** ma **jeden** warunek — wspólny porządek zapisów, ale **osobno dla każdej zmiennej**:
$$\forall_{x \in X} \ \forall_{w_1, w_2 \in OW \cap O\vert x} \left( \forall_{i} w_1 \mapsto_i w_2 \ \vee\ \forall_{i} w_2 \mapsto_i w_1 \right)$$
Jest to więc spójność sekwencyjna „per zmienna", bez żadnych gwarancji **między** zmiennymi.

**Spójność procesorowa** to **PRAM + podręczna** — oba powyższe warunki naraz.

### Hierarchia

Modele układają się w zagnieżdżenie, w którym model wewnętrzny jest **słabszy**:
$$\text{atomowa} \subset \text{sekwencyjna} \subset \text{przyczynowa} \subset \text{PRAM}$$
$$\text{procesorowa} = \text{PRAM} \cap \text{koherencja}$$
Idąc od zewnątrz: atomowa zachowuje czas rzeczywisty, sekwencyjna rezygnuje z niego na rzecz porządków lokalnych, przyczynowa rezygnuje ze wspólnego porządku zapisów, PRAM — z przyczynowości między procesami. Spójność podręczna stoi z boku: nie leży na tej osi, bo ogranicza się do pojedynczej zmiennej, a dopiero jej złożenie z PRAM daje spójność procesorową.

## Protokoły realizujące modele

Protokoły mają wspólny schemat: odczyt czyta lokalną replikę $M_i[x]$, zapis rozgłasza aktualizację $U(x,v)$, a **różnica leży w tym, który z dwóch dostępów blokuje i jakiego rodzaju rozgłaszania użyto**.

Wariant **fast-read** (odczyt natychmiastowy, zapis blokuje do rozgłoszenia) i **fast-write** (zapis natychmiastowy, odczyt blokuje, dopóki są niepotwierdzone własne zapisy) istnieje dla spójności sekwencyjnej i podręcznej. Dla procesorowej podaje się wariant fast-write. Modele przyczynowy i PRAM nie blokują nigdzie — zapis od razu aktualizuje replikę lokalną i rozgłasza dalej.

Kluczowy wzorzec, który warto umieć wypowiedzieć: **rodzaj rozgłaszania użytego w protokole odpowiada wprost realizowanemu modelowi**. Rozgłaszanie atomowe (totalne) daje spójność sekwencyjną, przyczynowe — przyczynową, FIFO — PRAM, a warianty indeksowane zmienną (`atomicx`, `FIFOx`, czyli porządek utrzymywany **osobno dla każdej zmiennej**) — spójność podręczną i procesorową. Model spójności danych jest więc **odbiciem porządku dostarczania komunikatów**, którym uspójniamy repliki; zob. [[RSO Z1 Komunikacja grupowa#Porządki dostarczania|zagadnienie 1]]. Pseudokody: [[RSO 02 Danocentryczne modele spójności#Protokoły realizujące modele|RSO 02]].

---
## Czego w prezentacjach nie ma

> [!todo] Luki w materiale
> **Definicja porządku przyczynowego** na operacjach (relacja $\rightarrow$) jest na slajdzie 31 osadzona jako pusta grafika bez warstwy tekstowej i **nie da się jej odczytać** — uzupełnić z podręcznika. **Modele przy dostępie synchronizowanym** (słaba, zwalniania, wejścia, zakresu) są wyłącznie wymienione, bez definicji. **Spójność ostateczna** (*eventual consistency*) nie występuje w prezentacjach w ogóle. Hierarchia modeli jest pokazana graficznie, ale **bez dowodów inkluzji**. Rejestr braków: [[RSO 02 Danocentryczne modele spójności#Braki w prezentacjach]]. Materiał uzupełniający: [[02 Danocentryczne modele spójności]], [[Algorytmy Rozproszone/Model Spójności]], [[Linearizability]].
