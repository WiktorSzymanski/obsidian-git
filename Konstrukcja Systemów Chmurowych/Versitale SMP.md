---
tags:
  - KonstrukcjaSystemówChmurowych
---
# Versitale SMP
---
> **vSMP** to programowa realizacja funkcji układu zarządzającego wieloma procesorami. Pomysł się nie przyjął, a projekt został zakończony.

- agregacja mocy obliczeniowej, pamięci i I/O
- spójny obraz dłuższego systemu wieloprocesowego
- emulacja BIOS-u i ACPI
- różna konfiguracja sprzętowa poszczególnych węzłów
- aktualnie max 128 węzłów, 1024 procesory (8192 rdzeni), 64 TB pamięci, ponad 1,5 TFLOPS
- zarządzanie pamięcią podręczną (spójność)
- migracja i replikacja bloków pamięci
- SMP jako model programowania (por. OpenMP, MPI, PVM)
- konkurencja ze storny superkomputerów i klastrów

#TODO Obrazek z prez

- połączenie realizowane jako przełączany InfiniBand
- programowy *chipset* sterujący - najdroższy element systemu SMP dużej skali 