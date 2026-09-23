---
tags:
  - obrona
up: "[[RSO 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 3
---
# 3. Modele spójności zorientowane na klienta
---
> [!warning] Zagadnienie nieobecne w prezentacjach
> Cała treść tej notatki pochodzi **spoza slajdów** — z wersji pierwszej [[03 Modele spójności zorientowane na klienta]] i notatek vaultu. W siedmiu prezentacjach (427 slajdów) jest na ten temat **jedno zdanie**: „modele spójności nastawione na klienta — uwzględnienie mobilności klienta". Wynik weryfikacji fraz i slajdów: [[RSO 03 Modele spójności zorientowane na klienta|RSO 03]].

## Po co osobna rodzina modeli

Modele **danocentryczne** opisują, co widzą **wszystkie procesy naraz**, i zakładają, że proces jest związany ze swoją repliką. Modele **zorientowane na klienta** (*client-centric*), zwane też **gwarancjami sesji** (Terry i in., system **Bayou**, 1994), odwracają perspektywę: nie mówią nic o globalnym stanie replik, a gwarantują, że **pojedynczy klient w ramach swojej sesji** zobaczy dane zgodne z własną historią interakcji — **nawet gdy przełącza się między serwerami**.

Motywacją są systemy ze **spójnością ostateczną** i klienci **mobilni**. Klient łączy się z różnymi replikami, z których każda może być nieaktualna; bez gwarancji sesji widziałby dane „cofające się w czasie" albo tracił własne zmiany. Zamiast więc synchronizować cały system, wymusza się warunek **tylko na tym serwerze, który obsługuje bieżące żądanie**.

## Założenia i aparat

Dane są replikowane na wielu serwerach, a replikacja jest **leniwa** — spójność ostateczna, propagacja np. przez anty-entropię. Każdy zapis ma globalnie unikalny identyfikator $WID$, zwykle parę $\langle \textit{serwer}, \textit{znacznik czasu} \rangle$. Przez $DB(S)$ oznacza się zbiór zapisów zastosowanych już na serwerze $S$.

Sesja klienta utrzymuje dwa zbiory: **write-set** $WS$ — identyfikatory zapisów wykonanych przez tego klienta — oraz **read-set** $RS$ — identyfikatory zapisów, które wpłynęły na wyniki jego odczytów. Pomocniczo $\textit{RelevantWrites}(S, r)$ to zapisy w $DB(S)$ wpływające na wynik odczytu $r$.

Cały mechanizm sprowadza się do jednego schematu: **przed obsłużeniem żądania serwer sprawdza, czy zawiera odpowiedni zbiór sesji**, a po obsłużeniu sesja jest aktualizowana.

## Cztery gwarancje

**RYW — Read Your Writes** (czytaj swoje zapisy): jeżeli klient wykonał zapis $w(x)$, to każdy jego późniejszy odczyt $r(x)$ realizuje serwer, który ten zapis **uwzględnia**. Warunek: odczyt na $S$ dozwolony, gdy $WS \subseteq DB(S)$. Klasyczne naruszenie: po zmianie hasła albo edycji profilu odświeżenie strony trafia na inny serwer i pokazuje starą wartość.

**MR — Monotonic Reads** (monotoniczne odczyty): jeżeli klient odczytał $x$ w pewnym stanie, to kolejne odczyty zwracają stan **co najmniej tak samo aktualny**, nigdy starszy. Warunek: odczyt na $S$ dozwolony, gdy $RS \subseteq DB(S)$; po odczycie $RS := RS \cup \textit{RelevantWrites}(S, r)$. Naruszenie: wiadomość widoczna w skrzynce znika po przełączeniu na inny serwer.

**MW — Monotonic Writes** (monotoniczne zapisy): jeżeli klient wykonał zapis $w_1$, a potem $w_2$, to **na każdym serwerze** $w_2$ zostaje wykonany **po** $w_1$. Warunek: zapis na $S$ dozwolony, gdy $WS \subseteq DB(S)$, a w $DB(S)$ $w_1$ poprzedza $w_2$. Naruszenie: aktualizacja dokumentu zastosowana zanim powstała jego wersja pierwsza.

**WFR — Writes Follow Reads** (zapisy następują po odczytach): jeżeli klient wykonał odczyt $r$, a potem zapis $w$, to na każdym serwerze $w$ jest umieszczony **po wszystkich zapisach, które wpłynęły na $r$**. Warunek: zapis na $S$ dozwolony, gdy $RS \subseteq DB(S)$, a $\textit{RelevantWrites}(r)$ poprzedzają $w$. Naruszenie: odpowiedź na forum pojawia się na serwerze, na którym oryginalnego postu jeszcze nie ma.

Warto zauważyć regularność: **RYW i MW pilnują write-setu, MR i WFR — read-setu**; **RYW i MR są sprawdzane przy odczycie, MW i WFR przy zapisie**. Wszystkie cztery są **niezależne** i można je dowolnie łączyć, ale nawet komplet nie daje spójności sekwencyjnej — dotyczą **jednej sesji**, nie relacji między wieloma klientami.

| Gwarancja | Sprawdzana przy | Warunek na serwerze |
|---|---|---|
| **RYW** | odczycie | $WS \subseteq DB(S)$ |
| **MR** | odczycie | $RS \subseteq DB(S)$ |
| **MW** | zapisie | $WS \subseteq DB(S)$, $w_1$ przed $w_2$ |
| **WFR** | zapisie | $RS \subseteq DB(S)$, zapisy istotne przed $w$ |

## Realizacja

Naiwnie klient przechowuje $WS$ i $RS$ jako zbiory identyfikatorów, a serwer sprawdza zawieranie — zbiory rosną wtedy **nieograniczenie**. Praktyczna realizacja (Bayou) używa **wektorów wersji**: serwer trzyma wektor $V_S$, którego składowa $V_S[k]$ to największy znacznik zapisu pochodzącego z serwera $k$ i zastosowanego na $S$ (zapisy z danego serwera stosuje się w kolejności). Sesja reprezentuje swoje zbiory wektorami $V_{WS}$ i $V_{RS}$, a warunek $WS \subseteq DB(S)$ staje się **dominacją po składowych** $V_{WS} \leq V_S$.

Gdy serwer warunku nie spełnia, są trzy wyjścia: klient **czeka**, klient **wybiera inny serwer**, albo serwer **pobiera brakujące zapisy** od innych replik. Dodatkowo MW i WFR wymagają, by serwery stosowały zapisy w porządku zgodnym z zależnościami — w praktyce **przyczynowym**, np. przy użyciu znaczników Lamporta w propagacji.

## Zestawienie z modelami danocentrycznymi

Modele danocentryczne opisują perspektywę **wszystkich procesów**, zakładają proces związany z repliką i kosztują **globalną synchronizację**. Modele zorientowane na klienta opisują perspektywę **jednej sesji**, zakładają klienta **mobilnego** i kosztują tyle, co porównanie wektora wersji. Pierwsze to [[RSO Z2 Danocentryczne modele spójności|zagadnienie 2]], drugie spotyka się w praktyce jako read-after-write w DynamoDB czy sesje przyczynowe w MongoDB.

---
## Czego w prezentacjach nie ma

> [!todo] Całe zagadnienie
> Prezentacje nie zawierają **niczego** poza wzmianką o mobilności klienta na slajdzie 26 wykładu 6. Brakuje założeń modelu, definicji RYW, MR, MW i WFR, przykładów naruszeń, implementacji wektorami wersji oraz relacji do modeli danocentrycznych. Materiał: [[03 Modele spójności zorientowane na klienta]], [[Algorytmy Rozproszone/Model Spójności#Klasyfikacja]], [[RSO 02 Danocentryczne modele spójności#Klasyfikacja modeli spójności replik]]; kontekst spójności ostatecznej: [[46 Ostateczna spójność - CRDT i typy chmurowe]].
