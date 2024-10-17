---
class: KSCh
---
# Xen
---
> **Xen** to zarządca wirtualizacji typu 1.

`dom0` - jedyna maszyna wirtualna z dostępem do sprzętu (Linux/BSD)
`domU` - nieuprzywilejowane domeny (maszyny wirtualne)
pełna wirtualizacja i parawirtualizacja

wiele trybów pracy:
- PV - paravirtualization
- HVM - hardware-assisted virtualization
- HVM + PV drivers
- PVHVM - HVM + PV drivers & interfaces (interrupts & timers) - lepsza optymalizacja
- PVH - paravirtualization + HVM interfaces
Wsparcie dla procesorów:
- Intel x86_64
- IA-32
- IA-64
- #TODO 

#TODO obrazek z przez


