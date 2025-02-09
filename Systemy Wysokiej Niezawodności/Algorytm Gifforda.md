---
tags:
  - SystemyWysokiejNiezawodności
---
# Algorytm Gifforda
---
>Algorytm **statycznego głosowania** mający na 
>celu pokonać ograniczenia atomowego zatwierdzania.
>
>Założenie algorytmu to przed wykonaniem operacji zbieramy głosy innych procesów w celu uzbierania **quorum**. Wyróżniamy dwa:
>- **read-quorum** oznaczane **$R$** - wymagane do operacji **read**
>- **write-quorum** oznaczane **$W$** - wymagane do operacji **write**
>  
>  Różne repliki mogą mieć różną ilość głosów. Istnieją dwa sposoby ich przydzielania:
>- na podstawie szybkości procesu
>- na podstawie niezawodności procesu

## Założenia
- model awarii _fail-recovery_ dla **procesów** i **kanałów**

## Struktury
- $VN_i$ - monotoniczny numer wersji $i$-tej repliki równy ilości dokonanych modyfikacji
- $V_i$ - liczba głosów $i$-tej repliki 
- $V$ - liczba głosów wszystkich replik 
$$V = \sum_{i}V_i$$
 - $M$ - większość głosów replik (_ang. majority_)
$$M = \left\lceil \frac{V+1}{2} \right\rceil$$
Aby zagwarantować poprawność $R$ i $W$:
- $W \geq M$ - tylko większość może zapisywać, eliminuje to możliwość powstania dwóch **quorum**
- $R + W > V$ - gwarancja, że zbiory procesów $R$ i $W$ mają część wspólną, to znaczy że przy zebranym **quorum** zawsze będzie aktualna replika

>[!hint]
>Zwany **statycznym** ponieważ $V_i$, $R$ i $W$ są stałe i niezależne od bieżącego stanu.

## Algorytm
1. $P_i$ składa $Lock\_Request$ do lokalnego zarządcy
2. Gdy blokada została założona $P_i$ wysyła komunikat $Vote\_Request$ do wszystkich procesów
3. Gdy $P_j$ odbiera $Vote\_Request$, składa $Lock\_Request$ do lokalnego zarządcy. Jeśli blokada została założona, odsyła do $P_i$ wersję swojej repliki $VN_j$ oraz liczbę głosów $V_j$.
4. $P_i$ decyduje o wykonaniu operacji w zależności od liczby uzbieranych głosów (_timeout_):
$$V_{read} = \sum_{k \in O} V_k \quad \text{gdzie }O\text{ oznacza zbiór numerów procesów, które przysłały głosy}$$
$$V_{\text{read}} \geq R \implies P_i \text{ zebrał quorum dla odczytu}$$
$$V_{\text{write}} = \sum_{k \in Q} V_k \quad \text{gdzie } Q = \{ k \in O : VN_k = VN_{\text{max}} \} \quad \text{i} \quad VN_{\text{max}} = \max \{ VN_j : j \in O) \}$$
  
$$V_{\text{write}} \geq W \implies P_i \text{ zebrał quorum dla zapisu}$$
> [!hint]-
 $V_{write}$ jest sumą wszystkich $V_k$ które odpowiedziały i ich numer wersji repliki $VN_k$ jest równy najbardziej aktualnemu numerowi wersji $VN_{max}$. On z kolei jest wyznaczany jako największy $VN_j$ dla każdego $j$ które jest w zbiorze procesów $O$, czyli tych które przysłały głosy. Jeszcze prościej jest to suma głosów najbardziej aktualnych replik, które odpowiedziały procesowi.

5. Jeśli $P_i$ nie zebrał **quorum**, wysyła $Release\_Lock$ do lokalnego zarządcy oraz
do wszystkich $P_k : k \in O$.
6. Jeśli $P_i$ zebrał **quorum**, sprawdza czy jego replika jest aktualna ($VN_i == VN_{max}$).
Jeśli $VN_i < VN_{max}$, aktualna kopia jest sprowadzana od dowolnego $P_k :k \in Q$.
1. Jeśli $P_i$ ma wykonać operację **read**, czyta lokalną kopię. Jeśli **write** – modyfikuje lokalną kopię oraz $VN_i$ i wysyła uaktualnienie do wszystkich $P_k : \in Q$ (czyli uaktualnia wyłącznie świeże/aktualne repliki). Wówczas $P_i$ wysyła $Release\_Lock$ do lokalnego zarządcy oraz do wszystkich $P_k : k \in O$.
2. Każdy $P_j$ otrzymując uaktualnienie modyfikuje lokalną replikę, a otrzymując $Release\_Lock$ zwalnia blokadę.

## Problemy
- co jeśli upadnie taka ilość procesów, że nie można uzbierać **quorum**
- lub oddzielą się od siebie poprzez partycjonowanie sieci

>[!hint]
>Tu może pomóc dynamiczne głosowanie.