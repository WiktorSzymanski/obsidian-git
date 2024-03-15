#SRC #Sem1 #PS

Deskryptor to liczba (`int`) do której jest dowiązanie symboliczne do i-węzła.

Lista deskryptorów plików procesu o identyfikatorze `[pid]` przechowywana jest w katalogu `/proc/[pid]/fd`. Katalog ten zawiera dowiązania symboliczne o nazwach odpowiadających numerom deskryptorów plików procesu. Każde takie dowiązanie symboliczne wskazuje na plik, z którym powiązany jest dany deskryptor.

W przypadku gniazd sieciowych, dowiązany plik zawiera etykietę typu (_socket_) oraz numer i-węzła danego [[Gniazda Sieciowe|gniazda sieciowego]]. I-węzły gniazd sieciowych przechowywane są w plikach w katalogu` /proc/net`.

#### Przykład
---
``` shell
$ ./server-tcp &
	[1] 21627 
$ ls -al /proc/21627/fd
	total 0
	dr-x------ 2 user users 0 Feb 17 11:02 . 
	dr-xr-xr-x 9 user users 0 Feb 17 11:02 .. 
	lrwx------ 1 user users 64 Feb 17 11:04 0 -> /dev/pts/0 
	lrwx------ 1 user users 64 Feb 17 11:04 1 -> /dev/pts/0 
	lrwx------ 1 user users 64 Feb 17 11:02 2 -> /dev/pts/0 
	lrwx------ 1 user users 64 Feb 17 11:04 3 -> socket:[69705] 
$ readlink /proc/21627/fd/3 
	socket:[69705]
$ cat /proc/21627/fdinfo/3
	pos: 0
	flags: 02
```

##### Stałe wartości deskryptorów:
>	0 - standardowe wejście
>	1 - standardowe wyjście
>	2 - wyjście diagnostyczne
	
Przy pomocy polecenia `netstat` możemy wyświetlić i-węzły w bardziej przyjazny sposób,