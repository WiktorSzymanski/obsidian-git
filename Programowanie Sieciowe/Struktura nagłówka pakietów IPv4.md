---
up:
tags: ProgramowanieSieciowe
---

#TODO - wszystko z prez

>`tos` -> Type of service
>	- flagi z pierwszymi 3 bitami 0 , bez priorytetu , 3 bity 1 powinno służyć do sterowania stąd priorytet
>	- są flagi któe oczekują, jak najszybszej, jak najbardziej niezawodnej, z jak najmniejszą niezawodnością
>	- wątpliwe czy routery biorą to pod uwagę/respekują chyba że ktoś w swojej sieci lokalnej/firmowej to ustawią

>`frag_off` -> Informacja który to jest obiekt z pociętego komunikatu, pozwala określić który to jest pakiet

>`daddr, saddr, id` -> trójka jednoznacznie identyfikująca pewn pakiet

> `ttl` -> time to live, 255 maksymalna długość, można po jej długości rozpoznać z jakiego _OS_ wysłany jest pakiet (jeśli używa on wartości domyślnych)

>`check` -> nadawca, odbiorca i każdy router musi obliczać sumę kontrolną, ponieważ mienia się nagłówek, (zmiana pola ttl sprawia że suma kontrolna się zmienia więc trzeba ją jeszcze raz obliczyć)

>po nagłówku mogą pojawić się opcje będące częścią nagłówka, nie ma on stałej wielkości nagłówka, stąd potrzebne jest pole `ihl` podające wielkość nagłówka

> aby dostać się do pola danych trzeba przesunąć się o `ihl` * 4 Bajtów.
> 5 * 4B -> najmniejszy nagłówek IPv4

>Nie dba o sumę kontrolną wysyłanych danych

> Net triality - amerykańska ustawa zabraniająca opóźniania streaming-u pewnych firm

#### Suma kontrolna nagłówka pakietu IPv4
---
Zaleta -> bardzo szybkie do wykonania