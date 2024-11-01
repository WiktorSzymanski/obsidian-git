---
up: "[[LDAP]]"
tags: ZarządzanieSystemamiKomputerowymi
---
# Directory Information Tree
---

>to hierarchiczna struktura organizacyjna, używana do przechowywania danych w usługach katalogowych opartych na [[LDAP]]. Organizuje dane w formie drzewa, gdzie każdy, gdzie każdy węzeł (_entry_) jest unikalny i może zawierać różne atrybuty, wartości, a także relację z innymi węzłami.
>
>Podobny do klasycznego systemu pliku, jednak nie posiada korzenia/katalogu głównego (_root_), każdy węzeł może zawierać dane i posiadać węzły potomne oraz jest możliwa odwrotna struktura nazw obiektów w drzewie.
>```
>/usr/local/bin/pico
>uid=voytek,ou=users,dc=put,dc=pl
>```

## ![[DN]]

## Struktura drzewa DIT

W strukturze drzewa DIT, każdy węzeł jest obiektem. Taki obiekt składa się z atrybutów określonego typ. Atrybuty mogą mieć wiele wartości, reprezentowanych jako łańcuch tekstowe.

![[Pasted image 20240525191530.png]]
