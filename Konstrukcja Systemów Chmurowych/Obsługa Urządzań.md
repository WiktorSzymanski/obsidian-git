---
tags:
  - KonstrukcjaSystemówChmurowych
up: "[[Wirtualizacja]]"
---
# Obsługa urządzeń
---
> **Obsługa urządzeń** odnosi się do różnych metod obsługi urządzeń w środowiskach wirtualnych. 

### Wyróżniamy trzy główne sposoby obsługi urządzeń:

 - **Emulacja** to programowa implementacja urządzeń, zastępująca sprzęt fizyczny bez przerywania pracy. Przykładami są wirtualne interfejsy sieciowe i wirtualne podsieci. Emulacja znanych urządzeń prowadzi do większej przenośności.
 
 - **Parawirtualizacja**  oferuje lepszą wydajność w porównaniu do emulacji.
 
 - **przekazywanie urządzań** ([[Device Passthrough]]) zapewnia jeszcze lepszą wydajność i niższe opóźnienia. Zazwyczaj brak możliwości współdzielenia urządzeń. Przykładami są przekazywanie PCI/PCIe.

## Problemy
Problemy obejmują obsługę DMA (translacja adresów) i zarządzanie przerwaniami (np. Message Signaled Interrupts – MSI).

## Wsparcie sprzętowe
Wsparcie sprzętowe obejmuje technologie takie jak:
- **Intel VT-d** (Virtualization Technology for Directed I/O)
- **Intel VT-c** (Virtualization Technology for Connectivity)
- **AMD-Vi** (poprzednio: **IOMMU**)