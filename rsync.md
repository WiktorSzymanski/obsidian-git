#SRC #Sem1 #ZSK 
- Protokół i program do synchronizacji zawartości dwóch (rozproszonych) plików
- Optymalizacja
#TODO wykład

### Algorytm Rsync
---
A - nadawca pliku fA
B - odbiorca posiadający kopie fB (index dolny?)

1. B dzieli plik fB na rozłączne części o równej wielkości S. Ostatni blok może być mniejszy.
2. Dla każdego bloku, B oblicza dwie sumy kontrolne:
	- słabą, cykliczną, 32-bitową sumę
	- silną, 128-bitową sumę MD5
3. B wysyła listę sum kontrolnych do A
4. A szuka w fA bloków o długości S, rozpoczynających się w dowolnym miejscu pliku (nie tylko wielokrotności S), które mają sumy kontrolne
#TODO

>_speedup_ - stosunek wielkości pliku do wysłanych danych w celu jego synchronizacji

#TODO cykliczna suma kontrolna prez 36/53

### Rsync + snapshots
---
1. Utworzenie pełnej kopii
2. Synchronizacja nowych modyfikacji
3. Utworzenie kopii z dowiązaniami (snapshot)
4. Kolejne synchronizacje modyfikacji usuwają dowiązania dla zmodyfikowanych plików w katalogu `/home`
5. #TODO 

#TODO obrazek prez 42/53

###### Laby
- synchronizacja dnych w jedną stronę
- wylicza sumy kontrole i na ich podstawie weryfikuje zmiany w pliku
- -t -> modyfikuje zmiane modyfikacji
	- w przypadku przełącznika `-t` jeśli nie zmieniły się daty modyfikacji, `speedup` (przyśpieszenie) jest znacząco wyższy niż w przypadku jego braku ( bez 65 z 2,395)
- kosztem tego jest przetwarzanie tych danych (liczenie sum kontrolnych)
- `dir/` -> odwołanie do zawartości katalogu, a nie katalogu samym w sobie
- usunięcie pliku wymaga dodanie przełącznika `--delete`. Jest tak ponieważ jeśli przeniesiemy pliki z katalogu i synchronizujemy katalog, to synchronizacja mogłaby nam wyczyścić kopie.
- `service rsyncd start`
- `read -s RSYNC_PASSWORD` pozwala zapisać zmienną środowiskową w nie jawny sposób w terminalu, a wstawienie spacji przed komendą nie zapisze instrukcji w historii terminala
- `-link-dest=../snap3` - tworzenie snapchota przy wykonywaniu `rsync`, minusem jest konieczność znania nazwy poprzedniego snapchota, przerwa w komunikacji może powodować że snapchot jest nie spójny
- `post-xfer exec = <skrypt.sh>` -> wykonanie skryptu po udanym `rsync`
