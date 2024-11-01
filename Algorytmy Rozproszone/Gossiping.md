---
tags:
  - AlgorytmyRozproszone
up:
---
#TODO
- Xerox chciał uruchomić replikowaną bazę danych w tysiącach lokalizacji
- Każda aktualizacja jest "wstrzykiwana" (ang. _injected_) w jednej lokalizacji i musi być propagowana do wszystkich innych lokalizacji
##### Problem:
	
##### Cel: 
- jak mnajmniej wymagać od systemu komunikacji


##### Dlaczego algorytmy epidemiczne? Dlaczego plotkowanie?
- Nawet w przypadku awarii, przez plotkowanie w końcu wszyscy otrzymają wiadomość
- Szybki, prosty, skalowalny

##### Rzeczywiste zastosowania:
- Amazon do wszybkiej transmisji informacji w Amazon S3
- Amazon Dynamo, awarie wykrywane dzięki mechanizmowi plotkowania
- BitTorent wymiana informacji na bazie plotkowania


### Model SIR dla systemu rozproszonego:
- Węzeł podatny (ang. _Suscepitable_)
- Węzeł zakażony (ang. )
- Węzeł wyleczony

### Algorytmy rozpraszania:


### Best effort:
- Nowe węzły mogą nie otrzymać rozgłoszenia
- Przy dużym obciążdeniu sieci nie wszystkie rozgłoszenia mogą dotrzeć

### Antyentropia:
- Ze względu na wysoki koszt tego rozwiązania, zaproponowano algorytmy optymalizujące