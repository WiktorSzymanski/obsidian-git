- wątek -> lekka forma współbieżności
- współbieżnie -> nieprzewidywalnie
- współbieżnie != jednocześnie, działania się ze sobą przeplatają
- procesy ciężkie izolujemy od siebie
- procesy naturalnie są odseparowane, trudność to ich współpraca
- wątki naturalnie współdzielą, trudność to ich odseparowanie
- wątki są równe, nie ma między nimi hierarchi
- w którymkoliwek wątku będzie funkcja `exit()`, wszystkie wątki w ramach tego procesu są zabijane
- w C kod wywyołujący `main()` po skończeniu tej funkcji wywołuje `exit()`
- dla każdego wątku powinniśmy albo `pthread_join()` albo `pthread_detatch()`
- współdzielą pamięć, ich stosy są w tej samej pamięci, ale mają osobne.

[coś]\_t oznacza że coś jest typem

pthread_create(pthread_t &id1, pthread_attrib_t NULL, jakaś funkcja) :
- wszystkie parametry wskaźnikowe
- paramert wyjściowy przez wskaźnik w argumentach
- pthread_[coś] wynikiem jest kod błędu, 0 oznacza jego brak
- pierwszy argument nie może być NULL, musimy podać jakiś adres
- drugi argument to wskaźnik na zmienną/obiekt opisujący właściwości wątku, NULL gdy chcemy argumenty domyślne
- da się dobić do pamięci innego wątku, ale jest to śliski temat
`pthread_join(id, &ret)`
	id -> identyfikator wątku
	ret -> parametr przechwytujący wynik, zmienna wskaźnikowa, dopuszczalny jest `NULL`
	Wykorzystywane w celu "połączenia" wąktu z wątkiem z którego wyszedł.
`pthread_detach(id)`
	id -> identyfikator wątku
	Jak nie planujemy nic z danym wątkiem robić powinniśmy wykonać dla nich tę funkcję.
`usleep(miktosekundy)`
	u ma przypominać grecką literę mikro (?)
Keyword `static`
	Jest tworzony jeden raz, statycznie  przy tworzeniu procesu/przy pierwszej aktywacji kodu bloku. Nie ma związku z częścią dynamiczną. Ma statyczną alokację w segmencie danych. W związku z tym że segment danych jest współdzielony przez wątki, zmienne statyczne są współdzielone.
Keyword `auto`
	nie trzeba go podawać jest domyślny, zmienna instnieje tylko na czas aktywacji danego bloku kodu.
	Zmienna `auto` jest tworzona na stosie.

Chodzi o to aby był ten sam mutex (`pthread_mutex_t`) dla wątków. Można to osiągnąć przez:
 - Utworzenie jej jako zmienną globalną
 - Jeśli używane tylko w danej funkcji to zmienna lokalna `static`

Przygotowanie mutex-u do użycia
``` C
pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;
```
`pthread_mutex_lock(&mtx)`
	blokuje mutex jeśli może, w przeciwnym razie czeka aż inny wątek zwolni mutex `mtx`

`pthread_mutex_unlock(&mtx)`
	odblokowanie mutex-a 
