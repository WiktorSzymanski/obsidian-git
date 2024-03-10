#SRC #Sem1 #PS #wykład 

Traktowane przez system jako plik specjalny. Aby korzystać z mechanizmu komunikacji internetowej wymagane jest utworzenie gniazda sieciowego.

Podstawowe gniazda sieciowe pozwalają na komunikację strumieniową (_TCP/IP_) i datagramową (_UDP/IP_).

##### Gniazda sieciowe w odniesieniu do warstwowego modelu internetowego
![[Pasted image 20240309155711.png]]

W języku _C_ można je utworzyć przy pomocy funkcji systemowej [[socket(2)]].

Dostęp do gniazd surowych i PF_PACKET ma tylko root.

Google zastąpił TCP własnym protokolem QUICK. Jeśli by mieli zaimplementować go jako surowe gniazdo to odpalenie googla by wymagało roota, lub poprosić wszystkich by dodali ich protokół. Zaimplementowali QUICK tak aby wykorzystywał UDP by to obejść.