---
up: "[[Systemy Plików]]"
class: ZSK
---
# Migawka (ang _snapshot_)
---

>to rodzaj kopi, której stan jest robiony w konkretnym momencie w czasie. Snapshot można porównać do zdjęcia przedstawiającego stan w momencie jego zrobienia. Kopie migawkowe są szybkie i efektywne, ponieważ kopia migawkowa nie polega na kopiowaniu plików, a zapisywaniu nowych zmian w innym miejscu. Stąd też nie zabiera dużo przestrzeni w porównaniu do pełnych kopii zapasowych. Ponieważ przedstawiają stabilny stan z łatwością można ja wykorzystać do przywrócenia danych w momencie wystąpienia awarii.

Poza kopiami zapasowymi i punktami przywracania, migawki mogą być skutecznie wykorzystywane do klonowania środowiska, w celu stworzenia takich samych stanowisk.

#### Zastosowania
- **Systemy plików** takie jak [[ZFS]] i [[Btrfs]] oferują wbudowany mechanizm tworzenia kopii migawkowych, używanych w celu ochrony danych.
- **Wirtualizacja** przez wirtualne maszyny takie jak VMware, Hyper-V czy VirtualBox wykorzystują migawki do zapisywania stanu maszyny wirtualnej.
- **Bazy danych** stosują migawki do tworzenia punktów przywracania danych. Przykładami takich systemów zarządzania bazami danych są Microsoft SQL Server czy Oracle.