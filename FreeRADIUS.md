#SRC #Sem1 #BSR #lab2 

Najpopularniejsza implementacja [[RADIUS]].

Serwer FreeRADIUS jest dostępny na licencji open-source i posiada szerokie możliwości konfiguracyjne oraz możliwości integracji z relacyjnymi bazami danych (np. MySQL lub PostgreSQL).

Jako bazę danych uwierzytelniających FreeRADIUS może wykorzystywać systemową bazę kąt użytkowników, własny plik konfiguracyjny kont użytkowników, usługę katalogową LDAP (w tym również Windows Active Directory) lub właśnie relacyjną bazę danych.

>zypper se freeradius
>rpm -ql freeradius-server | grep conf$
>rpm -ql freeradius-server | grep -w users

>/etc/raddb/users

We wdrożeniowej instalacji powinno się ustawić zabezpieczenia przed podsłuchaniem.

>service radius status
>/usr/sbin/radiusd -X -> run in debug mode

>radtest -x bob hello 127.0.0.1 10000 testing123

>ss -ltpn

#TODO zdjęcie
>int g4
>ip addr 10.1.0.100 255.255.0.0
>no shut