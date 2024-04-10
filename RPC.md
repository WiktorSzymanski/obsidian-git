#SRC #Sem1 #NPR 
#### Plik definujący interfejs
``` C
program Remote {
	version V1 {
		int rkill(int pid) = 7; /* 0 zarezerwowane dla ping */
	} = 1;
} = 0x20148084;
```
Rozszerzenie pliku to `.x`. Nazwa programu (`Remote`) nie może się powtarzać z żadną nazwą metody/funkcji zdalnej (`rkill`).
#### Generowanie 
``` bash
rpcgen -a <file>.x
```
> `-a` - dodaje trzy pliki: `server_template`, `client_sample`, `makefile`

Biblioteka `ltirpc` jest potrzebna do kompilacji. W makefile należy podmienić w `LDLIBS` `-lnsl` na `-ltirpc`