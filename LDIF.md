---
up: "[[LDAP]]"
class: ZSK
---
# LDAP Data Interchange Format
---

>to tekstowa reprezentacja węzłów drzewa [[DIT]], w formacie czytelnym dla człowieka. Można go wykorzystać do modyfikowania danych jak i umożliwia **wsadową zbiorczą modyfikację** poprzez eksport, modyfikacje danych i ich ponowny import. Dodatkowo można za jego pomocą dokonywać _backup_-u, migracji oraz konwersji do/z różnych baz danych. 

> [!Box]- Przykład
> 
> Dokument LDIF
> 
>``` LDAP
>dn: uid=voytek,ou=users,dc=put,dc=pl
>objectClass: account
>objectClass: posixAccount
>uid: voytek
>cn: Voytek Kovalsky
>uidNumber: 2123
>gidNumber: 110
>homeDirectory: /home/voytek
>userPassword: {crypt}vDdDYrjmydq96
>loginShell: /bin/bash
>```