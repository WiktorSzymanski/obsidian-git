---
tags:
  - obrona
up: "[[NPR 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 8
source: "slajdy–merged.pdf"
slajdy: "184–214"
---
# NPR 04. Podejście obiektowe — Java RMI
---
> Trzecie podejście do budowy systemów rozproszonych (po komunikatach i [[NPR 01 Zdalne wywoływanie procedur (RPC)|RPC]]): jednostką zdalnego dostępu jest **obiekt**, a zdalną operacją — **wywołanie metody**. Wykład omawia **Java RMI**: model systemu, sposoby przekazywania parametrów, architekturę stub-szkielet, kompletny cykl budowy aplikacji z kodem, **rejestr** (`rmiregistry`, `Naming`, `Registry`) oraz mechanizm **obiektów aktywowalnych** — jedyne w tym przedmiocie podejście do trwałości obiektu zdalnego.

---
## Model systemu
<sub>slajdy–merged.pdf, slajd 185</sub>

Klient nie widzi obiektu — widzi **interfejs**. Interfejs jest zdefiniowany w Javie i wywiedziony z `Remote`; po stronie klienta reprezentuje go **stub**, po stronie serwera **szkielet**.

![[npr-rmi-s185-model-systemu.png]]
<sub>Aplikacja kliencka woła metody interfejsu; stub i szkielet zamykają sieć. slajdy–merged.pdf, slajd 185</sub>

---
## Własności mechanizmu RMI
<sub>slajdy–merged.pdf, slajd 186</sub>

- Mechanizm RMI umożliwia tworzenie **obiektów zdalnych** — **brak bezpośredniego wsparcia dla tworzenia obiektów rozproszonych**.
- Jedyna forma zdalnego dostępu polega na wywoływaniu **metod wyspecyfikowanych w interfejsie** wywiedzionym (dziedziczącym) z `java.rmi.Remote`.
- Interfejs zdefiniowany jest **w języku implementacji** (nie w osobnym IDL, jak w Sun RPC czy CORBA).
- Obiekt może implementować **wiele interfejsów**.
- Ten sam interfejs może być implementowany w **wielu klasach** i występować w **wielu instancjach** każdej klasy.
- Interfejs traktowany jest **jak typ danych**.

Rozróżnienie „obiekt zdalny" a „obiekt rozproszony" jest tu kluczowe: obiekt zdalny **w całości leży na jednym węźle** i jest stamtąd udostępniany; obiekt rozproszony miałby stan rozrzucony po wielu węzłach. RMI daje tylko to pierwsze.

### Typ, wiązanie, trwałość
<sub>slajd 187</sub>

| Aspekt | W Java RMI |
|---|---|
| **Informacja o typie** obiektu (o zdalnym interfejsie) | dostępna **w czasie kompilacji** |
| **Wiązanie** obiektu | **jawne**, odbywa się **w czasie wykonania** |
| **Trwałość** obiektu | obiekt udostępniany przez `UnicastRemoteObject` ma charakter **przejściowy** — istnieje tylko w czasie działania serwera; dostępny jest mechanizm **obiektów aktywowalnych**, ale **brak bezpośredniego wsparcia dla utrwalania stanu** obiektu |

Napięcie między pierwszym a drugim wierszem jest charakterystyczne dla RMI: typ znany statycznie (więc kompilator sprawdza wywołania), ale **adres** znany dopiero w czasie wykonania (więc trzeba jawnie odpytać rejestr i rzutować wynik).

---
## Przekazywanie parametrów
<sub>slajdy–merged.pdf, slajdy 188–191</sub>

| Sposób | Warunek po stronie klasy | Co realnie trafia na drugą stronę |
|---|---|---|
| **przez wartość (kopię)** | klasa obiektu musi deklarować implementację `java.io.Serializable` | **kopia** obiektu |
| **przez referencję** | klasa obiektu musi implementować `java.rmi.Remote` | **zdalna referencja**, za którą udostępniany jest **proxy (stub)** |
| **przez kopiowanie i odtwarzanie** | — | **brak bezpośredniego wsparcia** |

### Przez wartość
<sub>slajd 189</sub>

![[npr-rmi-s189-przez-wartosc.png]]
<sub>Klient wysyła kopię obiektu; klasa obiektu implementuje `java.io.Serializable`. slajdy–merged.pdf, slajd 189</sub>

### Przez referencję
<sub>slajd 190</sub>

![[npr-rmi-s190-przez-referencje.png]]
<sub>Serwer dostaje proxy; wywołania na nim wracają do obiektu po stronie klienta. slajdy–merged.pdf, slajd 190</sub>

Warto zauważyć **odwrócenie ról**, które tu następuje. Gdy klient przekazuje jako parametr obiekt implementujący `Remote`, serwer dostaje **stub** i za jego pomocą wywołuje operacje **realizowane po stronie klienta** — to dokładnie **wywołanie zwrotne** z [[NPR 01 Zdalne wywoływanie procedur (RPC)#Warianty użycia RPC|NPR 01]], tylko wyrażone przez system typów zamiast przez osobny mechanizm. <sub>Opracowanie.pdf, s. 35</sub>

### Przez kopiowanie i odtwarzanie
<sub>slajd 191</sub>

![[npr-rmi-s191-kopiowanie-odtwarzanie.png]]
<sub>slajdy–merged.pdf, slajd 191</sub>

> [!note] Uzupełnienie z Opracowania
> Slajd 188 stwierdza tylko, że wsparcia brak; slajd 191 pokazuje diagram bez komentarza. Opracowanie wyjaśnia, jak taki efekt osiąga się ręcznie: przekazuje się obiekt jako parametr, z punktu widzenia Javy dostaje się **inny obiekt** jako odpowiedź, a następnie **wartości obiektu u klienta zastępuje się wartościami otrzymanymi od serwera**. <sub>Opracowanie.pdf, s. 36</sub>

---
## Architektura RMI
<sub>slajdy–merged.pdf, slajd 192</sub>

Trzy warstwy pod namiastkami:

![[npr-rmi-s192-architektura.png]]
<sub>Stub i szkielet, pod nimi obsługa referencji, transport RMI i TCP/IP. slajdy–merged.pdf, slajd 192</sub>

> [!note] Uzupełnienie z Opracowania
> Slajd nie mówi, że wybór protokołu transportowego jest sztywny. Opracowanie odnotowuje uwagę wykładowcy: **RMI bazuje na TCP i nie ma sposobu, żeby to zmienić**. Programista styka się wyłącznie z dwiema górnymi warstwami — klientem i serwerem udostępniającym obiekty. <sub>Opracowanie.pdf, s. 37</sub>

### Hierarchia klas w definicji zdalnego obiektu
<sub>slajd 193</sub>

![[npr-rmi-s193-hierarchia-klas.png]]
<sub>`RemoteObject` → `RemoteServer` → `UnicastRemoteObject`; obok, niezależnie, interfejs `java.rmi.Remote` specyfikujący zdalne metody. Klasa zdalnego obiektu dziedziczy z jednej gałęzi i implementuje drugą. slajdy–merged.pdf, slajd 193</sub>

Dwie gałęzie mają odmienne role: linia `RemoteObject` wnosi **implementację** (m.in. sensowne `equals`, `hashCode`, `toString` dla referencji zdalnych i mechanikę eksportu), interfejs `Remote` wnosi **specyfikację** tego, co wolno wołać zdalnie.

---
## Tworzenie aplikacji rozproszonej w Java RMI
<sub>slajdy–merged.pdf, slajdy 194–196</sub>

**1. Zdefiniowanie i implementacja odpowiednich klas** (w szczególności klas dla obiektów dostępnych zdalnie):
- zdefiniowanie **interfejsu pochodnego od `Remote`**,
- zdefiniowanie klasy wywiedzionej z `java.rmi.server.UnicastRemoteObject`, implementującej interfejs pochodny od `Remote`, **lub** użycie statycznej metody `exportObject` klasy `UnicastRemoteObject`.

**2. Kompilacja źródeł** (`javac`, `rmic`):

```
javac xxx.java   →  xxx.class
rmic  xxx        →  xxx_Stub.class
                    xxx_Skel.class     (we wczesnych wersjach Javy)
```

Krok `rmic` jest **zbędny od Javy 5** — namiastki są generowane dynamicznie w czasie wykonania.

**3. Udostępnienie wygenerowanego kodu klas** — trzy warianty: **wspólny system plików**, **kopia kodów klas** w różnych systemach plików, **udostępnianie kodu przez serwer WWW**.

**4. Uruchomienie aplikacji:**
- uruchomienie **`rmiregistry`** (_name server_),
- uruchomienie **serwera**: utworzenie zdalnych obiektów i ich **rejestracja** w `rmiregistry`,
- uruchomienie **klienta**: **zlokalizowanie** zdalnych obiektów (odwołanie do `rmiregistry`) i wywoływanie zdalnych metod.

![[npr-rmi-s196-client-server-side.png]]
<sub>Serwer rejestruje obiekt w rejestrze, klient lokalizuje go w rejestrze i wywołuje metodę; osobną ścieżką (serwer WWW) przesyłany jest kod klas. slajdy–merged.pdf, slajd 196</sub>

Kroki 3 i 4 są ze sobą powiązane: skoro obiekty przekazuje się **przez wartość**, klient musi skądś wziąć **bajtkod klasy** odbieranego obiektu — stąd ścieżka *code transfer* i serwer WWW na diagramie.

> [!note] Uzupełnienie z Opracowania
> Slajd 196 nie podkreśla, że rejestr jest osobnym bytem. Opracowanie: **RMI Registry funkcjonuje poza serwerem — „ma swój osobny serwer"**. <sub>Opracowanie.pdf, s. 39</sub>

---
## Kod aplikacji
<sub>slajdy–merged.pdf, slajdy 197–201</sub>

### Interfejs zdalny
<sub>slajd 197</sub>

Komunikacja pomiędzy serwerem (obiektem) a klientem jest określona przez definicję interfejsu pochodnego od `Remote`. Klasa zdalnego obiektu musi implementować ten interfejs.

```java
import java.rmi.Remote;
import java.rmi.RemoteException;

public interface Compute extends Remote {
    Object executeTask(Task t) throws RemoteException;
}
```

Każda metoda zdalna **musi deklarować `RemoteException`** — to sposób, w jaki system typów Javy wymusza obsługę awarii komunikacji, czyli realizuje „obsługę sytuacji wyjątkowych" ze slajdu 20 wykładu o RPC.

### Definicja klasy zdalnego obiektu (serwer)
<sub>slajd 198</sub>

```java
import java.rmi.*;
import java.rmi.server.*;
import compute.*;

public class ComputeEngine extends UnicastRemoteObject
                           implements Compute {
    public ComputeEngine() throws RemoteException {
        super();
    }

    public Object executeTask(Task t) {
        return t.execute();
    }
}
```

### Utworzenie i rejestracja zdalnego obiektu (serwer)
<sub>slajd 199</sub>

```java
public static void main(String[] args) {
    if (System.getSecurityManager() == null) {
        System.setSecurityManager(new RMISecurityManager());
    }
    String name = "//host:1099/Compute";
    try {
        Compute engine = new ComputeEngine();
        Naming.rebind(name, engine);
        System.out.println("ComputeEngine ok");
    } catch (Exception e) {
        System.err.println("ComputeEngine excep." + e.getMessage());
        e.printStackTrace();
    }
}
```

`1099` to domyślny port `rmiregistry`.

### Zdalne wywołanie metody (klient)
<sub>slajd 200</sub>

```java
import java.rmi.*;

public class launchComp {
    public static void main(String args[]) {
        if (System.getSecurityManager() == null) {
            System.setSecurityManager(new RMISecurityManager());
        }
        try {
            String name = "//" + args[0] + "/Compute";
            Compute comp = (Compute) Naming.lookup(name);
            CompArea task = new CompArea(...);
            BigDecimal area = (BigDecimal) (comp.executeTask(task));
            System.out.println(area);
        } catch (Exception e) {
            ...
        }
    }
}
```

Widać tu oba stwierdzenia ze slajdu 187 naraz: **rzutowanie** `(Compute)` jest możliwe, bo typ jest znany w czasie kompilacji; **`Naming.lookup`** jest konieczne, bo wiązanie jest jawne i następuje w czasie wykonania.

### Parametry przekazywanych obiektów
<sub>slajd 201</sub>

- Jeżeli przekazujemy obiekt **przez wartość**, to klasa tego obiektu musi implementować interfejs `Serializable`.
- Jeśli przekazujemy obiekt **przez referencję**, to klasa tego obiektu musi implementować interfejs `Remote`.

```java
public interface Task extends java.io.Serializable {
    Object execute();
}

public class CompArea implements Task {
    …
}
```

---
## Rejestr
<sub>slajdy–merged.pdf, slajd 202</sub>

| Element | Rola |
|---|---|
| **Rejestr** | **wiąże z nazwami** i udostępnia zdalne obiekty |
| **Tworzenie rejestru** | `rmiregistry [<port number>]` |
| **`LocateRegistry`** | umożliwia **tworzenie rejestru** (obiekty klasy `Registry`) — fabryka rejestrów |
| **`Registry`** | klasa **obiektu-rejestru** |
| **`Naming`** | **ułatwia korzystanie** z rejestru, umożliwiając jego specyfikację **w adresie URL** |

### `LocateRegistry`
<sub>slajd 203</sub>

```java
static Registry createRegistry(int port)               throws RemoteException
static Registry createRegistry(String host, int port)  throws RemoteException
static Registry getRegistry(int port)                  throws RemoteException
static Registry getRegistry(String host, int port)     throws RemoteException
```

`create*` **uruchamia** rejestr w bieżącej JVM, `get*` tylko **uzyskuje uchwyt** do już działającego.

### `Registry` — operacje na nazwach
<sub>slajd 204</sub>

Nazwa jest **prosta**, np. `"Compute"`:

```java
void     bind   (String name, Remote obj)
         throws RemoteException, AlreadyBoundException, AccessException
void     rebind (String name, Remote obj)
         throws RemoteException, AccessException
void     unbind (String name)
         throws RemoteException, NotBoundException, AccessException
String[] list   ()
         throws RemoteException, AccessException
Remote   lookup (String name)
         throws RemoteException, NotBoundException, AccessException
```

### `Naming` — te same operacje po URL
<sub>slajdy 205–206</sub>

Nazwa jest **adresem URL**, np. `"//host:1099/Compute"`, stąd dodatkowe wyjątki `MalformedURLException` i `UnknownHostException`:

```java
static void     bind   (String urlName, Remote obj)
                throws RemoteException, AlreadyBoundException, AccessException,
                       MalformedURLException, UnknownHostException
static void     rebind (String urlName, Remote obj)
                throws RemoteException, AccessException,
                       MalformedURLException, UnknownHostException
static void     unbind (String urlName)
                throws RemoteException, NotBoundException, AccessException,
                       MalformedURLException, UnknownHostException
static String[] list   (String urlName)
                throws RemoteException, AccessException,
                       MalformedURLException, UnknownHostException
static Remote   lookup (String urlName)
                throws RemoteException, NotBoundException, AccessException,
                       MalformedURLException
```

Różnica `bind` vs `rebind`: `bind` rzuca `AlreadyBoundException`, gdy nazwa jest zajęta; `rebind` nadpisuje. Dlatego w kodzie serwera (slajd 199) użyto `rebind`.

> [!note] Uzupełnienie spoza slajdów
> Praktyczna konsekwencja tego wyboru: przy **rejestrze zewnętrznym** (osobny proces `rmiregistry`) zabicie serwera **pozostawia obiekt w rejestrze**, więc ponowne uruchomienie z `bind` kończy się `AlreadyBoundException`. **Rejestr utworzony wewnątrz serwera** (`LocateRegistry.createRegistry`) temu zapobiega — ginie razem z serwerem, czyszcząc wpisy. <sub>Opracowanie.pdf, s. 60</sub>

---
## Obiekty aktywowalne
<sub>slajdy–merged.pdf, slajdy 207–208</sub>

| Pojęcie | Definicja |
|---|---|
| **Obiekt aktywowalny** | obiekt, którego **instancja może w danej chwili nie istnieć w żadnej maszynie wirtualnej**, pomimo istniejącej **referencji** do niego |

W momencie odniesienia do obiektu (czyli wywołania jednej z jego metod) instancja może zostać utworzona **w działającej lub specjalnie w tym celu uruchomionej** maszynie wirtualnej. Obiekt aktywowalny jest obiektem klasy pochodnej od `Activatable` (`java.rmi.activation`) lub obiektem jawnie obsługiwanym na potrzeby aktywacji przez odpowiednie **metody statyczne** klasy `Activatable`.

### Stany
<sub>slajd 208</sub>

| Stan | Znaczenie |
|---|---|
| **aktywny** | obiekt jest **eksportowany** (udostępniony zdalnie), a jego **instancja istnieje** w maszynie wirtualnej |
| **pasywny** | instancja **nie istnieje** (albo nie jest udostępniona zdalnie), ale **może zostać utworzona i wyeksportowana** w reakcji na odniesienie do niego |

**Tworzenie** obiektu aktywowalnego obejmuje dwa kroki: **rejestrowanie** obiektu w systemie aktywacji i **eksportowanie** obiektu (udostępnienie zdalnym klientom). **Uaktywnienie** obiektu zdalnego polega na uruchomieniu maszyny wirtualnej, w której następnie tworzona jest instancja tego obiektu.

### Wadliwa referencja
<sub>slajd 209</sub>

![[npr-rmi-s209-wadliwa-referencja.png]]
<sub>**Wadliwa referencja** (ang. _faulty reference_) składa się z **identyfikatora aktywacji** i **aktywnej referencji**. Dla obiektu **pasywnego** aktywna referencja jest pusta, a identyfikator wskazuje deskryptor aktywacji w aktywatorze; dla obiektu **aktywnego** aktywna referencja wskazuje instancję w JVM. slajdy–merged.pdf, slajd 209</sub>

Ta struktura tłumaczy, na czym polega cały mechanizm: klient trzyma **jedną referencję przez cały czas**, a to, czy wywołanie trafia prosto do instancji, czy najpierw uruchamia JVM, zależy od tego, które z dwóch pól jest wypełnione.

### Grupa aktywacji
<sub>slajd 210</sub>

Żeby mechanizm aktywacji mógł uaktywnić obiekt, **zarówno maszyna wirtualna, jak i sam obiekt muszą być odpowiednio opisane**.

- Opis **maszyny wirtualnej** związany jest z **grupą aktywacji**. Instancje obiektów należących do tej samej grupy tworzone są **w tej samej maszynie wirtualnej**.
- **Deskryptor grupy aktywacji** (`ActivationGroupDesc`) dostarcza informacji niezbędnych do **zidentyfikowania lub uruchomienia** właściwej JVM.

### Aktywator
<sub>slajdy 211–212</sub>

**Aktywator** (np. `rmid`) nadzoruje aktywację obiektów:
- utrzymuje **bazę informacji o obiektach aktywowalnych** — **deskryptory aktywacji** (klasa `ActivationDesc`) identyfikowane przez **identyfikatory aktywacji** (klasa `ActivationID`),
- **zarządza maszynami wirtualnymi**, w których udostępniane są obiekty.

Zasady funkcjonowania:
- **działa zawsze** podczas pracy systemu,
- **nie aktywuje (reaktywuje) obiektów, które są już aktywne**.

![[npr-rmi-s212-aktywator-deskryptory.png]]
<sub>Aktywator trzyma deskryptory grup aktywacji i deskryptory aktywacji; każda grupa odwzorowuje się na jedną JVM, w której powstają instancje jej obiektów. slajdy–merged.pdf, slajd 212</sub>

### Deskryptor i identyfikator aktywacji
<sub>slajdy 213–214</sub>

**Deskryptor aktywacji** dostarcza aktywatorowi informacji niezbędnych do utworzenia instancji aktywowanego obiektu. Przechowuje:

| Informacja | |
|---|---|
| **identyfikator grupy aktywacji** obiektu | wyznacza JVM |
| **nazwa klasy** obiektu | co utworzyć |
| **ścieżka do implementacji** obiektu | _codebase URL path_ |
| **dane inicjalizujące** | `MarshalledObject<?>` |

**Identyfikator aktywacji** zawiera informacje o obiekcie aktywowalnym:
- **zdalną referencję do aktywatora** obiektu,
- **unikalny identyfikator** obiektu.

Identyfikator aktywacji powstaje **w wyniku rejestracji** obiektu w systemie aktywacji, która odbywa się w jeden z trzech sposobów:
- przez wywołanie metody `Activatable.register`,
- przez użycie odpowiedniego **konstruktora klasy `Activatable`**, który rejestruje **i** eksportuje obiekt,
- przez wywołanie `Activatable.exportObject`.

Pole `MarshalledObject<?>` jest jedynym miejscem, w którym obiekt aktywowalny może odzyskać cokolwiek ze swojego wcześniejszego stanu — i dlatego slajd 187 mówi o **braku bezpośredniego wsparcia dla utrwalania stanu**: aktywacja przywraca *obiekt*, nie *jego stan sprzed dezaktywacji*.

---
## Braki w prezentacjach

> [!todo] Czego nie ma na slajdach 184–214
> - **Semantyka błędu RMI** — nigdzie nie powiedziano, którą z semantyk z [[NPR 01 Zdalne wywoływanie procedur (RPC)#Gwarancja wykonania — semantyka błędu|NPR 01]] realizuje RMI (jest to *co najwyżej raz*). Slajdy mówią jedynie, że metody deklarują `RemoteException`.
> - **Rozproszone zbieranie nieużytków** (_distributed garbage collection_, `Unreferenced`, dzierżawy) — mimo że jest to podstawowy problem obiektowego podejścia do systemów rozproszonych.
> - **CORBA i porównanie z RMI** — zagadnienie 8 wymienia podejście obiektowe ogólnie; prezentacja pokazuje wyłącznie RMI. `IDL`, ORB i pośrednik obiektowy nie występują.
> - **Zasady bezpieczeństwa** — kod używa `RMISecurityManager` bez wyjaśnienia, po co i jak wygląda plik polityki.
> - **`RemoteObject.equals`/`hashCode` dla referencji zdalnych** — slajd 193 pokazuje hierarchię, ale nie mówi, co ona wnosi.
> - Notatki w vaultcie [[Narzędzia Przetwarzania Rozproszonego/Wywoływanie metod zdalnych]] i [[Narzędzia Przetwarzania Rozproszonego/Istota Podejścia Obiektowego]] są **puste** — niniejsza notatka je zastępuje.
