---
class: KSCh, BSR
aliases:
  - KSCh
  - BSR
  - ZSR
tags:
  - BezpieczeństwoSystemówRozproszonych
  - ZarządzanieSystemamiRozproszonymi
  - KonstrukcjaSystemówChmurowych
---
# Docker
---
> **Docker** ujednolica proces wdrażania aplikacji na różnych systemach. Pozwala na przenośność aplikacji, ponieważ kontenery mogą działać na dowyolnym systemie wspierającym **Dockera**. Tworzy spójne środowisko adresujące problem *"u mnie działa"*.

- Kontenery dla pojedynczych aplikacji (LXC naśladował pełną wirtualizację OS)
- Infrastruktura standaryzująca tworzenie i zarządzanie kontenerami
- Infrastruktura dystrubucji #TODO 

Konfiguracja domyślna dockera -`/etc/docker/daemon.json`

## Zalety
- łatwość użycia
- separacja odpowiedzialności
	- development vs managment
- przenoścność (VM w systemach Mac OS X i Windows)
- Dostateczna izolacjia
- Wysoka efektywność
- Modularność
- #TODO 

## Jak działa
**Warstwowy system plików (OverlayFS)**
- Każdy obraz Docker składa się z wielu warstw
- Nowe warstwy są tworzone przy każdej zmianie w Dockerfile
- OverlayFS umożliwia ponowne wykorzystanie warstw, co przyśpiesza budowę obrazów
**Kontrolne grupy zasobów**
- Zarządzanie zasobami kontenerów, np. CPU, pamięcią, I/O
- Zapewnia izolacje zasobów między kontenerami
- Oganiczają zużycie zasobów przez poszczególne kontenery, chroniąc system gospodarza
**Podstawowe polecenia Docker CLI**
- `docker build` - buduje obraz
- `docker pull` - pobiera obraz z rejestru
- `docker run` - uruchamia kontener z podanego obrazu

## Elemnty infrasturktury
- Docker Engine - środowisko wykonawcze dla konenerów (serwer usługi, REST API)
- Dokcer Image - niemodyfikowalny wzorzec systemu plików dla kontenera
- Docker Registry - składnica wzorców
- #TODO 

## Docker Volume
- Wolumin danych kontenera
- Może przetrwać dłużej niż kontener
	- usunięcie kontenera 
	- aktualizacja (przebudowa) kontenera
- Można go dołączyć do wielu kontenerów
- Implementacja
	- katalog w systemie bazowym
	- sieciowy system plików

## Docker Images
Koncepcja wasrstwowej konstruckji systemu plików kontenera:
- systemu plików typu *union*
- uninanie duplikacji danych
- wykorzystanie mechanizmu *copy on write*
- szybkie tworzenie nowych kontenerów
- oszczędne #TODO

## Persistent Storage w Dockerze
- **Volumes** - przechowywanie danych, zarządzanie przez Docker
- **Bind Mounts** - bezpośredni dostęp do katalogów na systemie gospodarza
- **Kiedy używać**
	- **Volumes** - gdy potrzebujesz niezależności od systemu gospodarza
	- **Bind Mounts** - gdy aplikacja musi mieć dostęp do konkretynch plików na gospodarza

## Docker produkcyjnie
- Logowanie i monitoring
	- narzędzia takie jak ELK Stack, Prometheus, Grafana
- Orkiestracja
	- zarządzanie klastami kontenerów (Docker Swarm vs Kubernetes)
- Sieć w Dockerze
	- zarządzanie overlay networks, service discovery

