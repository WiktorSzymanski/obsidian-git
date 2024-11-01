---
up: "[[LDAP]]"
tags: ZarządzanieSystemamiKomputerowymi
---
# Odwołania do innych serwerów (ang. _referrals_)
---

to mechanizm wprowadzony w wersji 3, pozwalający serwerowi LDAP wskazać klientowi, że szukana przez niego informacja znajduje się na innym serwerze lub w innej części katalogu. Używane są w celu rozłożenia obciążenia i poprawienia skalowalności podczas zarządzania dużymi i złożonymi strukturami danych.

> [!Box]- #### Działanie
> 1. zapytanie klienta
> 2. zwrot adresu innego serwera
> 3. drugie zapytanie klienta
> 4. odpowiedź
> 
![[Pasted image 20240525120946.png]]

# Partycjonowanie przestrzeni nazw
---

Podczas partycjonowania trzeba przestrzegać pewnych ograniczeń:
- wszystkie obiekty w partycji muszą mieć wspólnego przodka
- wspólny przodek musi należeć do partycji

> [!Box]- Poprawne partycjonowanie
![[Pasted image 20240525121157.png]]

> [!Box]- Błędne partycjonowanie
![[Pasted image 20240525121252.png]]

Partycjonowanie wykonuje się poprzez mechanizm _referrals_ (`ref`). 

``` LDAP
dn: dc=put,dc=pl
objectClass: referral
objectClass: extensibleObject
dc: cs
ref: ldap://nazwa/dc=cs,dc=put,dc=pl/
```