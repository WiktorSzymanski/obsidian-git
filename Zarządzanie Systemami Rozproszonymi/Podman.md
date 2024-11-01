---
tags:
  - ZarządzanieSystemamiRozproszonymi
---
# Podman
---
> **Podman** to narzędzie do zarządzania kontenerami bez procesu dockerd (jest alternatywą do [[Docker|dockera]]).

## Zalety
- brak potrzeby uruchamiania deamona
- zwiększenie bezpieczeństwa, brak potrzeby uprawnień root

## Kompatybilność z Docker-em
- możliwość używania tych samych poleceń, np. `podman run`
- obrazy mogą być zaciągane z dockerhub-a
- możliwość budowania obrazów z dockerfile-ów