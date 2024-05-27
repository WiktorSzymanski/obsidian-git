---
up: 
class: ZSK
---
# OpenPKG
---

>to pewnego rodzaju meta dystrybucja, ponieważ nie jest przeznaczona do instalowania całego systemu operacyjnego, a raczej stworzenia pewnej nadbudowy na już istniejący system operacyjny. Postało aby stworzyć w miarę jednolite środowisku na wielu maszynach z innymi systemami operacyjnymi.

Wykorzystuje system operacyjny jako bazę (jądro i system plików), na którym buduje mechanizm zarządzania oprogramowaniem opartym na pakietach [[RPM]]. Całość trafia do odpowiedniego katalogu systemowego, nadbudowującego i czasami powielającego funkcje systemowe.

Korzystanie z OpenPKG nie wymaga uprawnień administratora, obsługuje skryptu startowe i pozwala na doinstalowywanie oprogramowania nieopakowanego przez OpenPKG.

#### Zalety
- Niezależność od systemu operacyjnego
- Izolowanie środowiska instalacji
- Aktywnie aktualizowany
#### Wady
- Mało popularny
- Mniejsza liczba pakietów w porównaniu do `apt` czy `yum`
- Zwiększenie złożoności zarządzania systemem, jeśli jest używane równocześnie z systemowym menadżerem pakietów
- Brak automatycznego mechanizmu zarządzania zależnościami