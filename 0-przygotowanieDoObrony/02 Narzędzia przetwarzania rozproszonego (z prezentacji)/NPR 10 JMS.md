---
tags:
  - obrona
up: "[[NPR 00 Indeks i mapowanie na prezentacje]]"
zagadnienie: 8
source: "slajdy–merged.pdf"
slajdy: "224–254"
---
# NPR 10. JMS (Java Message Service)
---
> Druga realizacja [[NPR 08 MOM i systemy kolejkowania komunikatów|MOM]] — przeciwieństwo [[NPR 09 ZeroMQ|ZeroMQ]]: **specyfikacja, nie produkt**, z pełnym brokerem, transakcjami i trwałością. Wykład (wg slajdów Cezarego Sobańca) omawia komponenty i funkcjonalność JMS, oba modele komunikacji, kompletny model programistyczny od fabryki połączeń po konsumenta, strukturę i typy wiadomości, a na końcu pięć mechanizmów niezawodności: **potwierdzenia, trwałość, priorytety, czas życia i kolejki tymczasowe**.

---
## Geneza
<sub>slajdy–merged.pdf, slajd 225</sub>

- **Opracowany w 1998.**
- **Pierwotny cel: dostęp do istniejących systemów kolejkowania wiadomości** (tzw. MOM — _Message Oriented Middleware_, np. IBM MQSeries).
- **Integralna część Java EE od wersji 1.3.**

To wyjaśnia charakter JMS: **nie jest to nowy system kolejkowania, tylko wspólny interfejs do systemów już istniejących**. Stąd bierze się rozdział na specyfikację i dostawcę.

---
## Komponenty JMS
<sub>slajdy–merged.pdf, slajd 226</sub>

| Komponent | Rola |
|---|---|
| **Dostawca JMS** (ang. _JMS provider_) | **implementacja interfejsów JMS**, administracja, sterowanie |
| **Klienci JMS** | aplikacje i komponenty **wysyłające i odbierające** komunikaty |
| **Wiadomości** | obiekty do **przenoszenia informacji** |
| **Obiekty zarządzania** (ang. _administered objects_) | **prekonfigurowane** obiekty na potrzeby zarządzania: **cele** (_destinations_) i **fabryki połączeń** (_connection factories_) |

**Obiekty zarządzania** są tu pojęciem kluczowym: nie tworzy ich programista w kodzie, tylko **administrator** poza aplikacją, a aplikacja je odnajduje. Dzięki temu zmiana kolejki czy adresu brokera nie wymaga rekompilacji — realizuje to **przezroczystość położenia**.

---
## Funkcjonalność JMS
<sub>slajdy–merged.pdf, slajd 227</sub>

- **Nieustanna, niezawodna, asynchroniczna komunikacja międzyprocesowa.**
- **Transakcyjna interakcja z dostawcą JMS.**
- **Modele komunikacji** (_messaging domains_): **punkt-punkt** (_point-to-point_) i **subskrypcji** (_publish/subscribe_).

Trzy przymiotniki w pierwszym punkcie to dokładnie trzy cechy MOM ze slajdu 3 — zob. [[NPR 08 MOM i systemy kolejkowania komunikatów#Cechy MOM|NPR 08]].

---
## Architektura
<sub>slajdy–merged.pdf, slajd 228</sub>

![[npr-jms-s228-architektura-jms.png]]
<sub>Narzędzie administracyjne wiąże (`Bind`) fabryki połączeń (CF) i cele (D) w przestrzeni nazw **JNDI**; klient JMS otrzymuje je przez wstrzykiwanie zasobu (`Inject Resource`), a z dostawcą JMS łączy się połączeniem logicznym. slajdy–merged.pdf, slajd 228</sub>

**JNDI** (_Java Naming and Directory Interface_) pełni tu rolę usługi nazewniczej — odpowiednik `rmiregistry` z [[NPR 04 Podejście obiektowe - Java RMI#Rejestr|NPR 04]] czy `portmap` z [[NPR 02 Sun RPC i standard XDR|Sun RPC]]. Adnotacja `@Resource(mappedName=…)` z kolejnych slajdów to właśnie odpytanie JNDI.

---
## Model punkt-punkt
<sub>slajdy–merged.pdf, slajd 229</sub>

- Wysyłanie i odbiór poprzez **dostęp do kolejki**.
- **Wiadomości pozostają w kolejce do czasu odbioru lub przedawnienia.**
- **Każda wiadomość ma 1 konsumenta.**
- **Brak ograniczeń czasowych** — odbiorca nie musi działać w chwili wysłania.

## Model subskrypcji
<sub>slajd 230</sub>

- **Każda wiadomość może mieć wielu konsumentów.**
- Wiadomości są **dostarczane do wszystkich aktualnych subskrybentów, a następnie usuwane**.
- **Nie można odczytać wiadomości sprzed subskrypcji.**
- **Trwała subskrypcja** (ang. _durable subscription_) — **odbiór wiadomości z okresu nieaktywności klienta**.

Trzeci punkt to ograniczenie paradygmatu publish/subscribe ze slajdu 5; czwarty jest odpowiedzią JMS na to ograniczenie — zob. [[#Subskrypcje trwałe]].

## Model odbioru wiadomości
<sub>slajd 231</sub>

| Rodzaj konsumpcji | Mechanizm |
|---|---|
| **synchroniczna** | **blokująca** metoda `receive()` z (ewentualnym) **ograniczeniem czasowym** |
| **asynchroniczna** | **odbiornik wiadomości** (ang. _message listener_) — asynchroniczne wywołanie metody `onMessage()` |

Wybór między nimi ma nieoczywiste konsekwencje dla potwierdzeń — zob. [[#Potwierdzenia wiadomości]].

---
## Model programistyczny
<sub>slajdy–merged.pdf, slajd 232</sub>

![[npr-jms-s232-model-programistyczny.png]]
<sub>`ConnectionFactory` tworzy `Connection`, ta tworzy `Session`, a sesja tworzy producenta, konsumenta i same wiadomości. Producent wysyła do `Destination`, konsument z niego odbiera. slajdy–merged.pdf, slajd 232</sub>

Łańcuch jest ściśle hierarchiczny: **każdy obiekt jest tworzony przez ten wyżej**, nigdy konstruktorem. To konsekwencja tego, że konkretne klasy należą do dostawcy, a aplikacja zna tylko interfejsy.

### Fabryki połączeń
<sub>slajd 233</sub>

| Interfejs | Zastosowanie |
|---|---|
| `ConnectionFactory` | **ogólna** — interfejs bazowy dla poniższych |
| `QueueConnectionFactory` | dla **kolejek** |
| `TopicConnectionFactory` | dla **tematów** |

### Destinations
<sub>slajd 234</sub>

| Typ | Zastosowanie |
|---|---|
| **`queue`** | komunikacja **punkt-punkt** |
| **`topic`** | model **subskrypcji** |

```java
@Resource(mappedName="jms/Queue")
private static Queue queue;

@Resource(mappedName="jms/Topic")
private static Topic topic;
```

### Połączenie
<sub>slajd 235</sub>

- **Reprezentacja wirtualnego połączenia z dostawcą JMS.**
- Połączenie jest **potrzebne do zainicjowania sesji**.
- Połączenie **musi być jawnie zamknięte** na końcu aplikacji (zwolnienie zasobów, m.in. otwartych sesji).
- Rozpoczęcie odbioru wiadomości: metoda `start()`; wstrzymanie: `stop()`.

```java
Connection connection = connFactory.createConnection();
connection.start();
...
connection.close();
```

Połączenie startuje **zatrzymane** — bez `start()` konsument nie dostanie nic. Jest to celowe: pozwala zarejestrować wszystkich konsumentów, zanim popłyną wiadomości.

### Sesje
<sub>slajd 236</sub>

- **Jednowątkowy kontekst** do tworzenia i odbioru wiadomości.
- Sesje tworzą: **wiadomości**, **producentów** (nadawców) i **konsumentów** (odbiorców).

```java
Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
```

**Pierwszy argument wskazuje, czy ma być tworzona transakcja.** Drugi to tryb potwierdzeń.

„Jednowątkowy" jest tu ograniczeniem, nie opisem: obiektów sesji **nie wolno dzielić między wątki**. Równoległość uzyskuje się, tworząc wiele sesji z jednego połączenia.

### Producent
<sub>slajd 237</sub>

```java
@Resource(mappedName="jms/Queue")
private static Queue queue;

MessageProducer producer = session.createProducer(queue);
producer.send(message);
```

Wysyłanie do **dowolnej** kolejki — producent bez przypisanego celu:

```java
MessageProducer producer = session.createProducer(null);
producer.send(queue, message);
```

### Konsument
<sub>slajd 238</sub>

```java
@Resource(mappedName="jms/Queue")
private static Queue queue;

MessageConsumer consumer = session.createConsumer(queue);
Message m1 = consumer.receive();       // blokuje bez ograniczenia
Message m2 = consumer.receive(1000);   // z limitem czasu [ms]
```

Odbiór asynchroniczny:

```java
MessageListener myListener = new AListener();
consumer.setMessageListener(myListener);
```

**Listener ma metodę `onMessage(Message)`. Musi obsługiwać wszystkie wyjątki** — nie ma komu ich przekazać, bo wywołanie następuje z wątku dostawcy.

### Selektory (filtry)
<sub>slajd 239</sub>

- Filtr jest **wyrażeniem zapisywanym jak warunki w SQL92**:

```sql
typ = 'wyniki' and id = '120'
```

- Wyrażenie odwołuje się do **właściwości wiadomości**.
- Filtr może być **parametrem tworzenia konsumenta** wiadomości.

Filtrowanie odbywa się **po stronie dostawcy**, więc odfiltrowane wiadomości nigdy nie trafiają do klienta. Ponieważ selektor działa na **właściwościach**, a nie na ciele wiadomości, to, po czym chce się filtrować, trzeba umieścić w nagłówku lub we właściwościach już przy wysyłaniu.

---
## Wiadomości
<sub>slajdy–merged.pdf, slajd 240</sub>

Wiadomości składają się z **nagłówka**, **listy właściwości** i **ciała**.

Standardowe właściwości (przechowywane w nagłówku):

| Właściwość | Znaczenie |
|---|---|
| `JMSMessageID` | identyfikator |
| `JMSDestination` | odbiorca |
| `JMSTimestamp` | znacznik czasowy |
| `JMSPriority` | priorytet |
| `JMSType` | typ |

### Typy wiadomości
<sub>slajd 241</sub>

| Typ | Ciało |
|---|---|
| `TextMessage` | wiadomość **tekstowa** (np. dokument XML) |
| `MapMessage` | zbiór **par nazwa-wartość** (`String` i typ prymitywny) |
| `BytesMessage` | **nieinterpretowany strumień bajtów** |
| `StreamMessage` | **strumień wartości** typów prymitywnych |
| `ObjectMessage` | **serializowalny obiekt Javy** |
| `Message` | **pusta** zawartość |

```java
TextMessage message = session.createTextMessage();
message.setText("Ala ma kota");
producer.send(message);
...
Message m = consumer.receive();
if (m instanceof TextMessage) { ... m.getText(); }
```

Zwróć uwagę na `instanceof` przed rzutowaniem: `receive()` zwraca `Message`, a odbiorca **nie wie z góry**, jaki typ przyjdzie.

`BytesMessage` jest dokładnie tym, co oferuje **jedyny** model [[NPR 09 ZeroMQ|ZeroMQ]] — JMS daje go jako jedną z sześciu opcji.

### Przeglądanie kolejki
<sub>slajd 242</sub>

- Interfejs **`QueueBrowser`**.
- Możliwość zastosowania **filtru**.
- **Nie można przeglądać tematów** (wiadomości znikają).

```java
QueueBrowser browser = session.createBrowser(queue);
Enumeration msgs = browser.getEnumeration();
while (msgs.hasMoreElements()) {
    Message m = (Message) msgs.nextElement();
    ...
}
```

Przeglądanie **nie konsumuje** wiadomości — to podgląd, nie odbiór. Niemożność przeglądania tematów wynika wprost z ich definicji ze slajdu 230: w temacie nie ma czego przeglądać, bo wiadomość znika natychmiast po rozesłaniu.

---
## Przesyłanie między systemami
<sub>slajdy–merged.pdf, slajd 243</sub>

![[npr-jms-s243-przesylanie-miedzy-systemami.png]]
<sub>Producent na jednym serwerze Java EE używa fabryki połączeń wskazującej na **zdalny** serwer; wiadomość trafia do kolejki na tamtym serwerze i stamtąd do konsumenta. slajdy–merged.pdf, slajd 243</sub>

Diagram pokazuje, gdzie naprawdę jest granica systemu: **fabryka połączeń decyduje, z którym dostawcą rozmawia klient**, a kod producenta wygląda identycznie jak przy kolejce lokalnej. To ilustracja przezroczystości położenia, którą wnoszą obiekty zarządzania.

---
## Niezawodność
<sub>slajdy–merged.pdf, slajd 244</sub>

Pięć mechanizmów: **potwierdzenia wiadomości** · **trwałość komunikatów** (awarie dostawców JMS) · **priorytety komunikatów** · **czas życia komunikatów** · **tymczasowe kolejki**.

### Potwierdzenia wiadomości
<sub>slajd 245</sub>

Potwierdzenie może następować **po odbiorze wiadomości**, **po przetworzeniu wiadomości** albo **po odbiorze potwierdzenia**. W sesji transakcyjnej **wiadomości są automatycznie potwierdzane po zakończeniu transakcji**, a **wycofanie transakcji → ponowne dostarczenie**.

Tryby potwierdzeń sesji nietransakcyjnej:

| Tryb | Zachowanie |
|---|---|
| `AUTO_ACKNOWLEDGE` | **po odbiorze** |
| `CLIENT_ACKNOWLEDGE` | **jawne potwierdzenie wszystkich** odebranych wiadomości w ramach sesji |
| `DUPS_OK_ACKNOWLEDGE` | **leniwe** potwierdzanie **z możliwością powstawania duplikatów** |

### Skutki wyboru trybu
<sub>slajd 246</sub>

- **Wiadomości niepotwierdzone przed końcem sesji są dostarczane ponownie** (przy kolejnym połączeniu).
- **Przetrzymywanie niepotwierdzonych wiadomości** dla trwałych subskrypcji.

Potwierdzenie **tylko przetworzonych** wiadomości zapewniają dwie kombinacje:

| Kombinacja | Wynik |
|---|---|
| odbiór **asynchroniczny** + `AUTO_ACKNOWLEDGE` | ✅ potwierdzenie po zakończeniu `onMessage()` |
| odbiór **synchroniczny** + `CLIENT_ACKNOWLEDGE` | ✅ potwierdzenie jawne, po przetworzeniu |
| odbiór **synchroniczny** + `AUTO_ACKNOWLEDGE` | ❌ **natychmiastowe potwierdzanie — przed przetworzeniem** |

Trzeci wiersz to pułapka: wiadomość potwierdzona w chwili powrotu z `receive()` **przepada**, jeśli przetwarzanie zaraz potem się nie powiedzie. Jest to dokładnie różnica między semantyką **_co najwyżej raz_** a **_co najmniej raz_** z [[NPR 01 Zdalne wywoływanie procedur (RPC)#Gwarancja wykonania — semantyka błędu|NPR 01]], a `DUPS_OK_ACKNOWLEDGE` jawnie wybiera tę drugą.

### Trwałość komunikatów
<sub>slajd 247</sub>

| Tryb | Zachowanie |
|---|---|
| **`PERSISTENT`** | każda wiadomość jest **rejestrowana w pamięci trwałej** — **tryb domyślny** |
| **`NON_PERSISTENT`** | wiadomość **może zostać utracona** w przypadku awarii dostawcy JMS — **większa wydajność** |

Własność trwałości może być ustawiana **dla producenta** wiadomości **lub dla pojedynczej wiadomości**:

```java
producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
producer.send(msg, DeliveryMode.NON_PERSISTENT, 3, 10000);
```

### Priorytety
<sub>slajd 248</sub>

- Priorytet może być ustawiany **dla producenta** lub **dla pojedynczej wiadomości**.
- **0 — najniższy priorytet, 9 — najwyższy, domyślnie 4.**
- **Priorytet określa preferencje i nie decyduje o bezwzględnej kolejności dostarczania.**

```java
producer.setPriority(5);
producer.send(msg, DeliveryMode.NON_PERSISTENT, 5, 10000);
```

### Czas życia
<sub>slajd 249</sub>

- Czas życia może być ustawiany **dla producenta** lub **dla pojedynczej wiadomości**.
- **Domyślnie wiadomości nie ulegają przedawnieniu.**
- **Czas życia 0 oznacza brak przedawniania.**

```java
producer.setTimeToLive(10000);
producer.send(msg, DeliveryMode.NON_PERSISTENT, 5, 10000);
```

Trzy ostatnie mechanizmy mają wspólną, czteroargumentową postać `send(msg, deliveryMode, priority, timeToLive)` — te same trzy parametry, które można też ustawić raz na producencie.

---
## Subskrypcje trwałe
<sub>slajdy–merged.pdf, slajdy 250–252</sub>

![[npr-jms-s250-trwale-subskrypcje.png]]
<sub>**Bez** subskrypcji trwałej: subskrybent istnieje tylko między `create` a `close`; wiadomości opublikowane poza tymi oknami (M3, M6) **przepadają**. slajdy–merged.pdf, slajd 250</sub>

![[npr-jms-s251-trwale-subskrypcje-2.png]]
<sub>**Z** subskrypcją trwałą: **subskrypcja** żyje od `create` do `unsubscribe`, niezależnie od tego, czy w danej chwili istnieje **subskrybent**; wiadomości z okresów nieaktywności zostają dostarczone po ponownym podłączeniu. slajdy–merged.pdf, slajd 251</sub>

Rozdzielenie **subskrypcji** od **subskrybenta** jest sednem mechanizmu: to subskrypcja jest bytem trwałym u dostawcy, a subskrybent — tymczasowym połączeniem do niej.

### Identyfikacja subskrypcji
<sub>slajd 252</sub>

Subskrypcję identyfikują **trzy** rzeczy: **identyfikator klienta**, **topic** i **nazwa subskrypcji**.

```java
connection.setClientID("MyId");
…
MessageConsumer topicSubscriber =
        session.createDurableSubscriber(myTopic, "MySub");
...
topicSubscriber.close();      // koniec subskrybenta, subskrypcja trwa
...
session.unsubscribe("MySub"); // koniec subskrypcji
```

Różnica między `close()` a `unsubscribe()` odpowiada dokładnie dwóm poziomom z diagramu.

---
## Transakcje
<sub>slajdy–merged.pdf, slajd 253</sub>

- **Grupowanie operacji wysyłania/odbioru w transakcji.**
- Metody **`Session.commit()`** i **`Session.rollback()`**.
- **Zatwierdzenie** oznacza **wysłanie** wyprodukowanych wiadomości i **potwierdzenie** odebranych.
- **Wycofanie** oznacza **usunięcie** wyprodukowanych wiadomości i **ponowne dostarczenie** wiadomości odebranych (**z pominięciem przedawnionych**).
- **Uwaga na zakleszczenia: wysłanie następuje po zatwierdzeniu.**

Ostatnie ostrzeżenie jest praktycznie najważniejsze: w jednej transakcji **nie wolno wysłać wiadomości i czekać na odpowiedź na nią**, bo wysłanie nastąpi dopiero przy `commit`, do którego nigdy nie dojdzie.

Własności transakcji JMS to te same własności, które opisuje [[SWN 09 Transakcje i atomowe zatwierdzanie]] — z tą różnicą, że tu transakcja obejmuje **operacje na kolejkach**, a nie na danych.

---
## Kolejki tymczasowe
<sub>slajdy–merged.pdf, slajd 254</sub>

```java
// utworzenie kolejki tymczasowej
TemporaryQueue tq = session.createTemporaryQueue();

// dołączenie kolejki tymczasowej do komunikatu
msg.setJMSReplyTo(tq);

// uzyskanie dostępu do kolejki tymczasowej (po odebraniu komunikatu)
tq = msg.getJMSReplyTo();
```

Jest to sposób zbudowania **komunikacji żądanie-odpowiedź** na mechanizmie jednokierunkowym: nadawca tworzy kolejkę zwrotną, dołącza ją do żądania, a odbiorca odsyła tam wynik. Innymi słowy — **RPC zaimplementowane na MOM**, z tą różnicą, że obie strony pozostają rozłączone w czasie.

---
## Braki w prezentacjach

> [!todo] Czego nie ma na slajdach 224–254
> - **Obsługa błędów i kolejki martwych listów** (_dead letter queue_) — co się dzieje z wiadomością, która po wielokrotnym ponownym dostarczeniu nadal nie może być przetworzona.
> - **Rozproszone transakcje XA** — slajd 253 opisuje wyłącznie transakcję lokalną w obrębie sesji; powiązanie z transakcją bazodanową (protokół dwufazowy, zob. [[SWN 09 Transakcje i atomowe zatwierdzanie]]) nie pada.
> - **Message-Driven Beans** — standardowy sposób konsumpcji JMS w Java EE, nieomówiony.
> - **JMS 2.0** (`JMSContext`, uproszczone API) — wykład opisuje wyłącznie API 1.1.
> - **Gwarancja kolejności dostarczania** — slajd 248 mówi, że priorytet jej nie określa, ale nie mówi, co ją określa.
