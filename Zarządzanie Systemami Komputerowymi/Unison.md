---
up: 
tags: ZarządzanieSystemamiKomputerowymi
---
# Unison
---

> to narzędzie do synchronizacji modyfikacji plików w obie strony, posiadające detekcję konfliktów. Stosuje protokół [[Rsync]] do przesyłania danych. Pracuje w środowisku heterogenicznym. Zapamiętywanie informacji o stanie każdego katalogu po udanej synchronizacji w celu rozróżniania operacji usuwania i dodawania nowych plików. W wypadku identycznych zmian po obu stronach żadne akcje nie są podejmowane.

- Lokalna detekcja zmian - zegary nie muszą być zsynchronizowane
#### Reguły synchronizacyjne
#REFACTOR
fi oznacza i‐tą wersję pliku f
H(f) oznacza znacznik czasowy pliku f

| Serwer A  | Serwer B  | Kopiowanie | Komentarz       |
| --------- | --------- | ---------- | --------------- |
| f1 —      | — —       | ->         | nowy plik       |
| f1 H(f1)  | f1 H(f1)  | ×          | stan spójny     |
| f2 H(f1)  | f1 H(f1)  | ->         | aktualizacja    |
| f2 H(f2)  | f3 H(f2)  | <-         | aktualizacja    |
| f4a H(f3) | f4b H(f3) | <->        | konflikt        |
| — H(f4)   | f4 H(f4)  | ->         | usunięcie pliku |
