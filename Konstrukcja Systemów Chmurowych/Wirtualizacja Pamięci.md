---
tags:
  - KonstrukcjaSystemówChmurowych
up: "[[Wirtualizacja]]"
---
# Wirtualizacja Pamięci
---
 - Podobna do klasycznej obsługi pamięci wirtualnej
 - Dodatkowy poziom wirtualizacji, poniżej wirtualizacji vOS
	 - pamięć aplikacji vOS
		 - wirtualna pamięć fizyczna vOS
		 - faktyczna pamięć fizyczna
 - Optymalizacja: tablice stron kopiowane z vOS w celu skrócenia odwzorowań (*ang. shadow pages*)
## Klasyczne odwzorowanie adresów pamięci
ostatnie odwzorowania **LPN** (*Logical Page Number*) -> **PPN** (*Physical Page Number*) przechowywane są w rejestrach **TLB** (*Translation Lookaside Buffer*)

![[Pasted image 20241228154520.png]]

- VMM: PPN -> MPN
- shadow page tables: LPN -> MPN (widoczne przez sprzęt)
- rejestry TLB zawierają ostatnie odwołania LPN -> MPN (przeładowanie podczas przełączanie na inny vOS). 
![[Pasted image 20241228154554.png]]
> Problem: Rejestry TLB były czyszcone co kwant czasu procesora więc tylko na chwilę mieliśmy te optymalizacje

##### Wsparcie sprzętowe drugiej generacji:
- AMD
	- Tagged TLB
	- Rapid Virtualization Indexing (RVI) (poprzednio: Nested Page Tables)
- Intel
	- Extended Page Tables
- wykorzystanie *huge pages*: 2MiB, 1GiB
- VMware tests: 42%, RedHat: 200% for OLTP, up to 600%

> Rozwiązuje to problem z czyszczonymi rejestrami.

MMU (Memory Menagement Unit) korzysta zarówno z odwzorowań LPN -> PPN jak i PPN -> MPN. Co za tym idzie znika konieczność korzystania z *shadow pages*.

![[Pasted image 20241228154634.png]]
### Zaawansowane zarządzanie pamięcią:
- opóźnianie alokacji pamięci
- [[CoW|copy on write]]
- zwalnianie nieużywanej pamięci
- deduplikacja stron pamięci - jeśli mamy odpalone wiele takich samych systemów to można wykorzystać tylko jednej załadowanej, nie trzeba ich ładować dla każdej VM.
	- VMware: *Transparent Page Shareing*
	- KVM: *Kernel SamePage Merging*
	- VirtualBox: *Page Fusion*
- kompresja stron pamięci
- sterownik pamięci typu *baloon* -> *overcommit* - vOS myśli że ma pewną ilość pamięci, a tym czasem jest ona dostępna dla hosta
![[VM-baloon.excalidraw.png]]