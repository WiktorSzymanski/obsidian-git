---
tags:
  - KonstrukcjaSystemówChmurowych
up: "[[Obsługa Urządzań]]"
---
# Device passthrough
---
**Device passthrough** (przekazywanie urządzeń) odnosi się do bezpośredniego przekazywania obsługi urządzeń do maszyny wirtualnej (VM), co zwiększa wydajność i zmniejsza opóźnienia.
Urządzenie znika z systemu bazowego i jest obsługiwane bezpośrednio przez VM. Dotyczy to między innymi urządzań USB i PCI/PCIe.

## Single Root Input/Output Virtualization (**SR-IOV**)
**SR-IOV** umożliwia jednemu urządzeniu dostarczenie wielu funkcji wirtualnych (VF) dla różnych VMs, wspierając współdzielenie magistrali i urządzeń PCIe. Pozwala też na przekazywanie ruchu sieciowego przez fizyczne karty sieciowe do czego przydatny jest standard Infiniband.

**vIOMMU** – emulacja **IOMMU** dla zagnieżdżonych **VMs**
## Problemy
- migracja VM
- PCI hotplug