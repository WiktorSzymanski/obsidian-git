#SRC #Sem1 #PS

Komenda systemowa `ip(8)` umożliwia manipulowanie wszystkimi tablicami routingu w systemie.

Odwołania do poszczególnych tablic mogą się odbywać z użyciem ich numerów lub nadanych im nazw zgodnie z odwzorowania mi zapisanym w pliku konfiguracyjnym `/etc/iproute2/rt_tables`.

###### Przykłady
---
``` shell
grep -v "^#" /etc/iproute2/rt_tables
	# Wyświetla tablice routingu i odpowiadające im numery

ip route show

ip route show table main

ip route show table local
```

> w przypadku komendy `ip route show` domyślnie pokazywana jest tablica `main`


###### Przykład konfiguracji _dodatkowej_ bramy domyślnej wybranej na podstawie adresu nadawcy/odbiorcy
``` Shell
ip route list

echo `1 my-tab` >> /etc/iproute2/rt_tables
	# Dodanie nowej tablicy do tablic routingu

ip route add 10.0.0.0/24 dev eth1 src 10.0.0.2 table my-tab
#TODO prez 11/32

```

>Polityki trasowania mogą być użyteczne aby rozłożyć ruch sieciowych. 

>_zob._ `ip-rule(7)`

