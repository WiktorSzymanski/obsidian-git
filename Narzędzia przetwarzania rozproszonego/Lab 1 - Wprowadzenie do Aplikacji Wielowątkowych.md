- wątek -> lekka forma współbieżności
- współbieżnie -> nieprzewidywalnie
- współbieżnie != jednocześnie, działania się ze sobą przeplatają
- procesy ciężkie izolujemy od siebie
- procesy naturalnie są odseparowane, trudność to ich współpraca
- wątki naturalnie współdzielą, trudność to ich odseparowanie

[coś]\_t oznacza że coś jest typem

pthread_create(pthread_t &id1, pthread_attrib_t NULL, jakaś funkcja) :
- wszystkie parametry wskaźnikowe
- paramert wyjściowy przez wskaźnik w argumentach
- pthread_[coś] wynikiem jest kod błędu, 0 oznacza jego brak
- pierwszy argument nie może być NULL, musimy podać jakiś adres
- drugi argument to wskaźnik na zmienną/obiekt opisujący właściwości wątku, NULL gdy chcemy argumenty domyślne