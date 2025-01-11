---
tags:
  - KonstrukcjaSystemówChmurowych
up: "[[Wirtualizacja w procesorach z rodziny x86]]"
---
# Parawirtualizajca
---
- Zmodyfikowany vOS, świadomy wirtualizacji
- Instukcje krytyczne -> jawne wywołania nadzorcy
- Nadzorca: zarządzanie pamięcią, obsługa przerwań
- Lepsza wydajność przez brak konieczności wykonywania wielu operacji zamiast jednej w przypadku instrukcji wrażliwych

![[Pasted image 20241228153716.png]]
#### Przykłady
- Xen
- MS Hyper-V
- VMware Tools
- VirtualBox Guest Additions
