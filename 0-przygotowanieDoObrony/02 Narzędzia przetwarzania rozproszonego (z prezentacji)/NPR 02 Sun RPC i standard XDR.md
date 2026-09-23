---
tags:
  - obrona
up: "[[NPR 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 8
source: "slajdy–merged.pdf"
slajdy: "47–76"
---
# NPR 02. Sun RPC i standard XDR
---
> Konkretna realizacja mechanizmu z [[NPR 01 Zdalne wywoływanie procedur (RPC)|NPR 01]]. Wykład pokazuje **Sun RPC** jako kompletne narzędzie — od pliku `.x` z opisem interfejsu, przez generator `rpcgen`, po **standard XDR** (RFC 1014) realizujący konwersję kanoniczną. Trzon XDR to para pojęć **potok** (gdzie dane leżą i w którą stronę idą) i **filtr** (procedura konwersji), z podziałem filtrów na proste, złożone i pochodne.

---
## Właściwości Sun RPC
<sub>slajdy–merged.pdf, slajd 48</sub>

- **trójwymiarowa identyfikacja procedur** — (nr programu, nr wersji, nr procedury),
- protokół RPC oparty na protokołach warstwy transportowej stosu **TCP/IP** (TCP lub UDP),
- **wiązanie dynamiczne** (`portmap` lub `rpcbind`),
- **kanoniczny format reprezentacji danych** (XDR),
- opis interfejsu w języku **C-podobnym** i przetwarzanie przez narzędzie **`rpcgen`**.

Każda z tych pozycji to wybór jednej z opcji omówionych ogólnie w [[NPR 01 Zdalne wywoływanie procedur (RPC)]]: wiązanie dynamiczne zamiast statycznego, konwersja kanoniczna zamiast bezpośredniej, identyfikacja procedur realizująca warstwę SELECT.

---
## Tworzenie aplikacji — pełny cykl
<sub>slajdy–merged.pdf, slajdy 49–52</sub>

**1. Przygotowanie pliku z opisem interfejsu** w języku RPC (języku narzędzia `rpcgen`), zawierającego:
- definicje struktur danych dla przekazywanych parametrów,
- specyfikację procedur zdalnych: numer programu, numer wersji i numer procedury, typy argumentów, typ zwracanej wartości.

**2. Wygenerowanie przez `rpcgen` plików do kompilacji w języku C:**
- plik nagłówkowy z definicjami stałych,
- pliki z **namiastką klienta** i **namiastką serwera**,
- pliki zawierające **szkielet procedur zdalnych** (fragment programu serwera) i **szkielet programu klienta** dla potrzeb przetestowania procedur zdalnych,
- plik do zarządzania kompilacją (`Makefile`),
- plik z **funkcjami XDR** do konwersji zdefiniowanych typów danych.

**3. Uzupełnienie lub modyfikacja wygenerowanych plików:**
- uzupełnienie plików zawierających szkielety procedur zdalnych,
- uzupełnienie/modyfikacja programu klienta,
- uzupełnienie/modyfikacja pliku `Makefile`.

**4. Kompilacja programów.**

**5. Testowanie działania procedur zdalnych** — ewentualne przygotowanie aplikacji w wersji **zwartej** (nierozproszonej) w celu przetestowania działania samych procedur.

**6. Przygotowanie programu klienta** korzystającego z procedur zdalnych w sposób wynikający z potrzeb aplikacji.

Punkt 2 realizuje „przetwarzanie interfejsu" ze slajdu 30 wykładu o RPC — lista generowanych plików pokrywa się z listą ogólną jeden do jednego.

> [!note] Uzupełnienie spoza slajdów
> Praktyka uruchamiania: `rpcgen -a plik.x` generuje dodatkowo trzy pliki — `server_template`, `client_sample` i `makefile`. Do kompilacji potrzebna jest biblioteka `libtirpc`: w `Makefile` należy w zmiennej `LDLIBS` podmienić `-lnsl` na `-ltirpc`. Nazwa programu nie może się pokrywać z nazwą żadnej procedury zdalnej. <sub>[[Narzędzia Przetwarzania Rozproszonego/RPC]]</sub>

---
## Specyfikacja procedur zdalnych
<sub>slajdy–merged.pdf, slajd 53</sub>

Ogólna postać pliku `.x`; `program` i `version` są słowami kluczowymi:

```c
program NAZWA_PROGRAMU {
  version NAZWA_WERSJI1 {
    typ nazwa_proc1(typ_arg) = nr_proc;
    typ nazwa_proc2(typ_arg) = nr_proc;
    …
  } = numer_wersji;
  version NAZWA_WERSJI2 {
    typ nazwa_proc1(typ_arg) = nr_proc;
    typ nazwa_proc2(typ_arg) = nr_proc;
    …
  } = numer_wersji;
} = numer_programu;
```

Zagnieżdżenie odwzorowuje trójwymiarową identyfikację: **program → wersja → procedura**. Jeden program może udostępniać równocześnie wiele wersji interfejsu — to mechanizm zgodności wstecznej.

### Zakresy numerów programów
<sub>slajd 54</sub>

| Zakres | Przeznaczenie |
|---|---|
| `0x00000000 – 0x1FFFFFFF` | numery **standardowe**, zdefiniowane (przydzielane) przez firmę Sun |
| `0x20000000 – 0x3FFFFFFF` | przydzielone przez **użytkownika** |
| `0x40000000 – 0x5FFFFFFF` | zarezerwowane dla aplikacji, które **generują numery programów dynamicznie** |
| `0x60000000 – 0xFFFFFFFF` | **zarezerwowane** |

> [!note] Uzupełnienie z Opracowania
> Slajd nie podaje szerokości pola. Numery programów są **32-bitowe** — stąd taki, a nie inny zapis zakresów. <sub>Opracowanie.pdf, s. 22</sub>

### Przykład — zdalne usunięcie procesu
<sub>slajdy 55–57</sub>

Wersja 1 — jeden argument całkowity, sygnał domyślny:

```c
program REMOTE_KILL {
   version DEFAULT_SIGNUM {
      int rkill(int) = 1;
   } = 1;
} = 0x20000001;
```

Wersja 2 — numer sygnału podawany jawnie, parametry spakowane w **strukturę**:

```c
struct rkill_params {
   int pid;
   int signum;
};

program REMOTE_KILL {
   version SPECIFIED_SIGNUM {
      int rkill(rkill_params) = 1;
   } = 2;
} = 0x20000001;
```

Wersja 2 w wariancie z **dwoma osobnymi argumentami**:

```c
program REMOTE_KILL {
   version SPECIFIED_SIGNUM {
      int rkill(int, int) = 1;
   } = 2;
} = 0x20000001;
```

Ten sam **numer programu** i **numer procedury**, różne **numery wersji** — obie wersje mogą być udostępniane równolegle przez ten sam serwer.

---
## Standard XDR
<sub>slajdy–merged.pdf, slajd 58</sub>

| Cecha | Treść |
|---|---|
| **Opis standardu** | RFC 1014 |
| **Reprezentacja** | kanoniczna, oparta na formacie **IEEE** |
| **Język opisu** | **deklaratywny** język opisu struktur danych, zbliżony do języka C |
| **Koncepcja konwersji** | oparta na **potokach** i **filtrach** |

| Pojęcie | Definicja |
|---|---|
| **Potok** | miejsce **przechowywania** danych w formacie XDR |
| **Filtr** | procedura **konwersji** pomiędzy własnym formatem maszyny a formatem kanonicznym (w obu kierunkach) |

### Potok
<sub>slajd 59</sub>

Potok jest miejscem składowania danych XDR, a dokładniej **środkiem dostępu** do nich w celu:
- **zapisu** — potok **kodujący** (_encoding_),
- **odczytu** — potok **dekodujący** (_decoding_).

| Rodzaj potoku | Gdzie leżą dane |
|---|---|
| **na standardowym wejściu-wyjściu** | na otwartym pliku |
| **w pamięci** | w obszarze pamięci opisanym przez **adres i rozmiar w bajtach** |
| **komunikatów (rekordów)** | na **strumieniu danych**, zdefiniowanym wraz z procedurami obsługi tego strumienia (zapis, odczyt) |

### Filtr
<sub>slajd 60</sub>

Filtr jest procedurą konwersji, służącą do realizacji dostępu do danych w potoku w celu zapisu (kodującą) lub odczytu (dekodującą) — **zależnie od kierunku działania potoku**.

| Rodzaj filtru | Zastosowanie |
|---|---|
| **filtr prosty** | konwersja **typów prostych** |
| **filtr złożony** | konwersja **typów złożonych** (np. tablicowych, wskaźnikowych) |
| **filtr pochodny** | **połączenie innych filtrów** w ramach jednej procedury filtrującej |

![[npr-xdr-s61-potoki-i-filtry.png]]
<sub>Ten sam filtr obsługuje oba kierunki — to **potok** określa, czy filtr koduje, czy dekoduje. slajdy–merged.pdf, slajd 61</sub>

To najważniejsza własność projektowa XDR: programista pisze **jedną** procedurę na typ, a nie osobno serializację i deserializację.

---
## Filtry proste
<sub>slajdy–merged.pdf, slajd 62</sub>

Filtry istnieją dla: **typów prostych** — `bool_t`, `char`, `short`, `int`, `long` (również bez znaku), `float`, `double`; **typu wyliczeniowego** (`enum`); **typu `void`**.

Ogólna postać procedury filtrującej, na przykładzie `unsigned int`:

```c
bool_t xdr_u_int(XDR* xdrs, unsigned int *ptr);
```

---
## Filtry złożone
<sub>slajdy–merged.pdf, slajdy 63–67</sub>

| Konwertowany typ | Filtry |
|---|---|
| tablice **bajtów** o ustalonej lub zmiennej wielkości | `xdr_opaque`, `xdr_bytes` |
| tablice **elementów określonego typu** o ustalonej lub zmiennej wielkości | `xdr_vector`, `xdr_array` |
| **łańcuchy znaków** | `xdr_string`, `xdr_wrapstring` |
| **typy wskaźnikowe** | `xdr_reference`, `xdr_pointer` |
| **unie** | `xdr_union` — w praktyce dla statycznie zdefiniowanych typów unijnych stosowany jest **filtr pochodny** |

### Ogólna postać
<sub>slajd 64</sub>

```c
xdr_typ(XDR* xdrs, ptr, …, [xdrproc_t elproc]);
```

Dwie reguły rządzą postacią parametrów:
- **typ statyczny** (ustalona zajętość pamięci) → wskaźnik `ptr` ma postać `char *ptr`;
  **typ o zmiennej wielkości** (konieczność dynamicznej alokacji pamięci) → `char **ptr`;
- jeśli **typ podstawowy** (w przypadku tablic, wskaźników) nie jest z góry określony, konieczne jest wskazanie **filtru XDR do konwersji elementu** typu podstawowego (`elproc`).

### Tablice
<sub>slajd 65</sub>

```c
xdr_array(xdrs, arrp, sizep, maxsize, elsize, elproc)
  XDR *xdrs;
  char **arrp;
  u_int *sizep, maxsize, elsize;
  xdrproc_t elproc;

xdr_vector(xdrs, arrp, size, elsize, elproc)
  char *arrp;
  u_int size, elsize;   // pozostałe parametry — j.w.

xdr_bytes(xdrs, sp, sizep, maxsize)
  char **sp;            // pozostałe parametry — j.w.

xdr_opaque(xdrs, cp, cnt)
  char *cp;
  u_int cnt;
```

Rozróżnienie jest konsekwentne: `xdr_array`/`xdr_bytes` (zmienna wielkość) biorą `char **` i **wskaźnik** na rozmiar `sizep` oraz ograniczenie `maxsize`; `xdr_vector`/`xdr_opaque` (ustalona wielkość) biorą `char *` i rozmiar przez wartość.

### Łańcuchy znaków
<sub>slajd 66</sub>

```c
xdr_string(xdrs, sp, maxsize)
  XDR *xdrs;
  char **sp;
  u_int maxsize;

xdr_wrapstring(xdrs, sp)
  xdr_string(xdrs, sp, MAXUN.UNSIGNED);
```

`xdr_wrapstring` **nie wymaga podania rozmiaru** (liczby znaków łańcucha) — jest opakowaniem `xdr_string` z maksymalnym możliwym ograniczeniem. Dzięki temu ma sygnaturę zgodną z `xdrproc_t` i może być przekazywany jako `elproc`.

### Struktury wskaźnikowe
<sub>slajd 67</sub>

```c
xdr_reference(xdrs, pp, size, proc)
  XDR *xdrs;
  char **pp;
  u_int size;
  xdrproc_t proc;

xdr_pointer(xdrs, objpp, objsize, xdrobj)
  XDR *xdrs;
  char **objpp;
  u_int objsize;
  xdrproc_t xdrobj;
```

**`xdr_pointer` w przeciwieństwie do `xdr_reference` interpretuje wskaźnik pusty**, co umożliwia obsługę **rekurencyjnych struktur danych** — bez tego nie dałoby się zakodować listy ani drzewa, bo nie byłoby jak zapisać końca. To praktyczna odpowiedź na **problem przetaczania** z [[NPR 01 Zdalne wywoływanie procedur (RPC)#Problem przetaczania|NPR 01]].

---
## Filtr pochodny
<sub>slajdy–merged.pdf, slajd 68</sub>

Filtr pochodny to zwykła procedura `bool_t`, która po kolei woła filtry składowych i **przerywa przy pierwszym niepowodzeniu**:

![[npr-xdr-s68-filtr-pochodny.png]]
<sub>Definicja struktury po lewej, odpowiadający jej filtr pochodny po prawej. slajdy–merged.pdf, slajd 68</sub>

```c
bool_t xdr_struktura(XDR *xdrs, struktura *objp) {
    if (!xdr_int  (xdrs, &objp->x)) return FALSE;
    if (!xdr_long (xdrs, &objp->y)) return FALSE;
    if (!xdr_char (xdrs, &objp->c)) return FALSE;
    if (!xdr_short(xdrs, &objp->s)) return FALSE;
    return TRUE;
}
```

Taki filtr jest dokładnie tym, co `rpcgen` generuje automatycznie dla każdej struktury zadeklarowanej w pliku `.x`.

---
## Zarządzanie pamięcią
<sub>slajdy–merged.pdf, slajd 69</sub>

- Przekazanie **adresu pustego wskaźnika** (typu `char**`) przy konwersji **dekodującej** spowoduje **dynamiczną alokację pamięci** przez filtr XDR.
- **Zwolnienie** obszaru dynamicznie zaalokowanej pamięci przez filtr XDR musi nastąpić **w programie aplikacyjnym**.
- Ogólna postać funkcji zwalniania pamięci:

```c
xdr_free(xdrproc_t proc, char* objp);
```

`xdr_free` bierze **ten sam filtr**, którym dane były dekodowane — filtr wie, jak obejść strukturę, więc potrafi ją też zwolnić rekurencyjnie.

---
## Tworzenie potoków
<sub>slajdy–merged.pdf, slajdy 70–72</sub>

```c
// potok na standardowym wejściu-wyjściu
void xdrstdio_create(XDR *xdrs, FILE *file, enum xdr_op op);

// potok w pamięci
void xdrmem_create(XDR *xdrs, char *addr, u_int size, enum xdr_op op);

// potok komunikatów
void xdrrec_create(XDR *xdrs,
                   u_int sendsize, u_int recvsize,
                   char *handle,
                   int (*readit)(char*, char*, int),
                   int (*writeit)(char*, char*, int));
```

Zasady użycia:
- **Ustalenie kierunku potoku musi nastąpić po jego utworzeniu** (np. `xdrs->x_op = XDR_ENCODE`). Zwróć uwagę, że `xdrrec_create` — w odróżnieniu od dwóch pozostałych — **nie przyjmuje** parametru `op`.
- Rozmiary buforów (`sendsize`, `recvsize`) mogą mieć wartość **0**, co oznacza przyjęcie wartości domyślnych.
- Gdy konieczne jest **opróżnienie bufora wyjściowego** (wysłanie danych) lub **zapełnienie bufora wejściowego**, wywoływana jest odpowiednia funkcja (`writeit`, `readit`) z trzema parametrami: **uchwytem** `handle`, **adresem bufora** i **liczbą bajtów** do zapisu/odczytu.

### Potoki komunikatów — granice rekordów
<sub>slajd 72</sub>

| Operacja | Funkcja |
|---|---|
| oznaczenie **końca rekordu** (przy zapisie) | `xdrrec_endofrecord(XDR *xdrs, int sendnow)` |
| **pominięcie reszty rekordu** (przy odczycie) | `xdrrec_skiprecord(XDR *xdrs)` |
| sprawdzenie **zakończenia strumienia** (przy odczycie) | `xdrrec_eof(XDR *xdrs)` |

Potok komunikatów jest jedynym, który wprowadza pojęcie **granicy rekordu** — potrzebne, bo strumień (np. TCP) sam z siebie granic komunikatów nie zachowuje.

---
## Odwzorowanie typów: XDR → C
<sub>slajdy–merged.pdf, slajdy 73–76</sub>

`rpcgen` tłumaczy deklaracje z pliku `.x` na definicje C. Reguła ogólna: do każdej definicji dokładany jest `typedef`, żeby nazwa typu była używalna bez słowa `struct`/`enum`.

| XDR (`rpcgen`) | język C |
|---|---|
| `const MAX = 512;` | `#define MAX 512` |
| `enum COLOR { red = 1, green = 2, blue = 4 };` | `enum COLOR { red = 1, green = 2, blue = 4 };`<br>`typedef enum COLOR COLOR;` |
| `struct ST { int a; int b; };` | `struct ST { int a; int b; };`<br>`typedef struct ST ST;` |

### Unie
<sub>slajd 75</sub>

Unia XDR jest **rozróżnialna** (ma dyskryminator) i tłumaczy się na **strukturę** zawierającą dyskryminator oraz zagnieżdżoną unię C:

```c
/* XDR */                        /* C */
union UN switch (int d) {        struct UN {
   case 1: int a;                   int d;
   case 2: char b;                  union {
   default: short c;                   int a;
};                                     char b;
                                       short c;
                                    } UN_u;
                                 };
                                 typedef struct UN UN;
```

### Tablice i łańcuchy
<sub>slajd 76</sub>

Nawiasy rozróżniają tablicę o **ustalonej** wielkości (`[…]`) od tablicy o **zmiennej** wielkości (`<…>`):

```c
/* XDR */                        /* C */
typedef int tabf[10];            typedef int tabf[10];

typedef int tabv<10>;            typedef struct {
                                     u_int tabv_len;
                                     int  *tabv_val;
                                 } tabv;
```

Tablica o zmiennej wielkości staje się więc **parą: długość + wskaźnik** — dokładnie tym, czego oczekuje `xdr_array`.

Podobnie tablice bajtów — typem bazowym jest wówczas `char`:

```c
typedef opaque btf[10];
typedef opaque btv<10>;
```

Łańcuchy znaków:

```c
typedef string s10<10>;   // najwyżej 10 znaków
typedef string sbo<>;     // bez ograniczenia
```

---
## Braki w prezentacjach

> [!todo] Czego nie ma na slajdach 47–76
> - **Postać komunikatu RPC na drodze** (nagłówek `call`/`reply`, pola uwierzytelniania, kody odrzucenia) — slajdy opisują wyłącznie warstwę języka i konwersji.
> - **`portmap`/`rpcbind` w działaniu** — mechanizm jest wymieniony na slajdzie 48 jako cecha, ale nie pokazano protokołu rejestracji ani zapytania; ogólny schemat wiązania dynamicznego jest w [[NPR 01 Zdalne wywoływanie procedur (RPC)#Wiązanie klienta z serwerem|NPR 01]].
> - **Uwierzytelnianie w Sun RPC** (`AUTH_NONE`, `AUTH_SYS`, `AUTH_DES`) — nie występuje wcale.
> - **Wyrównanie do 4 bajtów** — podstawowa reguła kodowania XDR z RFC 1014 nie pada na żadnym slajdzie, mimo że tłumaczy, czemu `xdr_opaque` dopełnia dane.
> - **Sposób obsługi `xdr_union`** — slajd 63 stwierdza tylko, że „w praktyce stosowany jest filtr pochodny", bez pokazania jak.
