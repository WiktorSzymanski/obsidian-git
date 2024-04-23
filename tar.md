#SRC #Sem1 #ZSK 
##### Przesłanie plików przy użyciu strumienia `tar` i `netcat`
###### Na urządzaniu docelowym 
``` Shell
netcat -l <port> | tar xvf -
```

###### Na urządzeniu źródłowym 
``` Shell
tar cvf - <src_dir> | netcat <host> <port>
```