---
tags:
  - KonstrukcjaSystemówChmurowych
up: "[[Docker]]"
---
# Docker Images
---
>Koncepcja warstwowej konstrukcji systemów plików kontenera:
>- systemy plików typu union
>- unikanie duplikatów danych
>- wykorzystanie machanizmu [[CoW|copy on write]]
>- oszczędne gospodarowanie przestrzenią dyskową 
>	- współdzielenie danych między kontenerami
>	- współdzielenie danych między #TODO
>- #TODO

#### Przykład konfiguracji warstw
runtime
PHP
nginx
updates
openSUSE
- aktualizacje poszczególnych warstw
- wielokrotne wykorzystywanie warstw
- buforowanie warstw pamięci 