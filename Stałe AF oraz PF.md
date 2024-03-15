#SRC #Sem1 #PS

Prefiks `AF` odnosi się do rodziny adresowej _(ang. address family_), a prefiks `PF` odnosi się do rodziny protokołu (_ang. protocol family_) — domena komunikacyjna.

Podział taki wprowadzono na przypadek gdyby pojedynczy protokół wspierał wiele sposobów adresacji — stała `PF` niezbędna byłaby wówczas do utworzenia gniazda sieciowego, a stała `AF` używana byłaby w strukturze adresowej.

W rzeczywistości przypadek taki nigdy nie wystąpił i stałe AF mają te same wartości co stałe PF dla poszczególnych protokołów.

#### Przykład
---
``` shell
$ cat ./socket.h | tr -s " " "\t" | cut -f 2,3 | grep "_INET"
	PF_INET   2
	PF_INET6  10
	AF_INET   PF_INET
	AF_INET6  PF_INET6
```