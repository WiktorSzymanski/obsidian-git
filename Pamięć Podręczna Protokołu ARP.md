#SRC #Sem1 #PS 

Pamięć poręczna protokołu `ARP` (ang. _ARP cache_) przechowuje odwzorowania adresów `IP` w adresy [[Warstwa Łącza Danych|warstwy łącza danych]] (np. adresy `MAC`), które protokół ten dotychczas rozwiązał.

Każdy wpis w pamięci podręcznej `ARP` zawiera następujące informacje:
- adres `IPV4`
- adres warstwy łącza danych (domyślnie jest to `ether`)
- maska adresu warstwy łącza danych
- flagi: C ("complete"), M ("permanent"), P ("publish")
- interfejs sieciowy, poprzez który nastąpiło odwzorowanie

>zob. `arp(7)`,  `apr(8)`, `if_arp.h`

Zawartość pamięci podręcznej protokołu `ARP` dostępna jest także w pliku `/proc/net/arp`. Polecenie `arp -n` wyświetla nam zawartość właśnie tego pliku, w przyjaznym dla użytkownika formacie.

Oznaczenie `(incomplete)` ma miejsce gdy została podjęta próba uzyskania adresu `MAC` dla danego `IP` ale nie rozwiązany został jego adress`MAC`

Zapytania `ARP` działają tylko w sieci lokalnej.

>Dodanie adresu z flagą "permament" jest jedynym sposobem na obronienie sie przed ARP Spoofingiem.

`arp -s <ip> <mac>`  - ustawienie rekordu arp