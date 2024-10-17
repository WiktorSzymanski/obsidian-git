---
up: "[[Domain Model]]"
---
# Aggregate *(z ang. Agregat)*
---
> to określenie używane do przedstawienia modelowanych "granic" i zasad **bezpośrednio powiązanych** obiektów. Prościej mówiąc to grupa **powiązanych obiektów** uważanych za **pojedynczą jednostkę** w kwestii zmian informacji. Oznacza to, że jakakolwiek zmiana obiektu, znajdującego się w agregacie, uznawana jest za zmianę **stanu agregatu**.

**Agregat** to abstrakcyjne określenie często nie występujące we rzeczywistej [[Domain|domenie]], lecz jest przydatne w [[Domain Model|modelu domeny]]. Posiada on:
- Grupę obiektów, mogą być one typów [[Entity]] i/lub [[Value Object]]
- Relacje jakie te obiekty formują (zależności, zawieranie się w sobie itp.)
- Stałe/niezmienne *(ang. Invariants)*, które są wymuszane pomiędzy tymi obiektami

Zasadniczo, agregat definiuje "granicę" oddzielającą obiekty wewnątrz niego od tych z poza niego. W agregacie musi znajdować się **pojedynczy obiekt**, który wchodzi w interakcje z obiektami z poza agregatu. Taki obiekt nazywa się **korzeniem agregatu** *(ang. Aggregate Root)*.

## Modelowanie Agregatu
---
Podczas modelowania agregatu znaczące jest aby przestrzegać następujących zasad:
- Obiekty poza agregatem **nie mogą** posiadać referencji do obiektów wewnątrz agregatu.
- Obiekty poza agregatem **mogą** posiadać referencje do **korzenia agregatu**
- Obiekty wewnątrz agregatu **mogą** posiadać referencje do siebie nawzajem
- Obiekty wewnątrz agregatu **mogą** posiadać referencje do **korzenia innego agregatu**
- **Korzeń agregatu** jest odpowiedzialny za utrzymywanie stałych/niezmiennych *(ang. Invariants)*
- Jeśli **korzeń agregatu** zostanie usunięty, obiekty wewnątrz agregatu również **są usuwane**, ponieważ nie ma jak się do nich odwołać

#### Przykład
---
Załóżmy, że _"zarządzasz_ **planem podróży** _pasażera"_, który składa się z kilku **etapów**. Etap to pojedynczy **lot** z miejsca początkowego do miejsca docelowego, a każdy etap obsługiwany jest przez określony lot. Kombinacja lotów zapewnia **pasażerowi** dotarcie z miejsca początkowego do docelowego. Istnieje kilka możliwych sposobów ustalenia granic agregatu:

- Jeden agregat reprezentuje **pasażera**. Inny agregat reprezentuje **plan podróży** ze wszystkimi **etapami** i **lotami**.
- Jeden agregat reprezentuje **pasażera** i **plan podróży**. Inny agregat reprezentuje każdy **etap** i odpowiedni **lot**.
- Jeden agregat reprezentuje **pasażera**, **plan podróży** i **etapy**. Inny agregat reprezentuje każdy **lot**.

W przypadku problemu, który musisz rozwiązać, _"zarządzanie planem podróży dla klienta"_, najlepszą odpowiedzią byłaby ostatnia opcja wymieniona powyżej z następujących powodów:

- **Pasażer**, **plan podróży** i **etapy** stanowią jedną jednostkę w odniesieniu do zmian danych.
- Istnieją dobrze zdefiniowane relacje: **Plan podróży** należy do **pasażera**; **Plan podróży** składa się z **etapów**.
- Istnieją niezmienniki: początek pierwszego etapu i miejsce docelowe ostatniego nigdy się nie zmieniają.
- Plan podróży jest tym, co jest ważne dla zewnętrznego obiektu, więc jest to **korzeń agregatu**
- Odniesienia do **pasażera** i poszczególnych **etapów** z obiektów zewnętrznych nie mają sensu i nie są dozwolone.
- **Etapy** mają odniesienia do **lotów** (inny agregat), które mogą się zmieniać bez wpływu na plan podróży.