#SRC #Sem1 #AR #TODO

Model spójności definiuję gwarancje udzielenia przez system w odniesieniu do uporządkowania operacji zleconych przez użytkownika (klienta, aplikację).

Zarządzanie spójnością odbywa się przez protokół spójności lub koocherencji. Jego celem jest uspójnienie replik jak i także transparentność:
- Spójność silna - z punktu widzenia klienta system zachowuje się tak, jakby nie było replikacji -> niesatysfakcjonująca efektywność
- Spójność słaba - system dostarcza pewnych gwarancji co do stanu replik => ograniczona intuicyjność wykorzystania środowiska
- Spójność z jawną synchronizacją - wsparcie użytkownika pozwalające na uzyskanie...

Aby zachować spójność replik, należy zapewnić, że wszystkie konfliktowe operacje są wykonywane w tej samej kolejności na każdej replice.

Konfliktowe operacje:
- read-write
- write-write

...

### Klasyfikacja
---
>Modele klasy _data-centric_
>	określają restrykcje na porządek, w któym operacje są postrzegane na poszczególnych serwerach oraz w procesach klentów.

>Modele klasy _client-centric_
>	określają restrykcje odnośnie obrazu danych (stanu repliki obiektów) oparte wyłącznie na historii interakcji poszczególnych klientów z systemem.

Powód rozróżnienia modeli _data-centric_ i _client-centric_:
- W modelach _data-centric_ pojęcia "serwer " i "klient" (użytkownik, aplikacja) są tożsame
- W modelach _client-centric_ zakłada się możliwość przełączenia użytkownika pomiędzy serwerami

...obrazki


### Definicja Modeli Spójności
---
...

##### Definicja uszeregowania legalnego
...

##### Definicja historii
...

##### Definicja spójności atomowej
Oznacza to dużo blokad, wszystkie operacje będą musiał być synchronizowane jako że obraz historii powinien być taki sam dla każdego procesu.
...

##### Rozluźnienie spójności atomowej
Porządek nie musi być zachowany w czasie rzeczywistym
...


##### Definicja spójności sekwencyjnej
Repliki muszą mieć zmiany w tej samej kolejności.
...
