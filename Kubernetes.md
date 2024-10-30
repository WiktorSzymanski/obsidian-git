---
tags:
  - ZarządzanieSystemamiRozproszonymi
---
# Kubernetes
---
>**Kubernetes** to platforma do zarządzania aplikacjami kontenerowymi w klastrze komputerów. Został zaprojektowany przez Google i oddany społeczności open-source w 2014 roku. Zapewnia **automatyzację**, **skalowanie** i **zarządzanie** aplikacjami w kontenerach. Rozwiązuje problemy związane z zarządzaniem setkami lub tysiącami kontenerów na wielu serwerach. W **Kubernetesie** wszystkie zasoby są definiowane za pomocą plików [[YAML]].

Nie trzyma żandych twardych referencji. "Trzyma" rzeczy luźno po nazwach. Stąd wszystkie nazwy muszą być **unikatowe**.

**Deployment** w Kubernetes automatyzuje proces wdrażania i zarządzania Podami. Główne funkcje deploymentu to:
- Automatyczne odtwarzanie *Pod*-ów w razie awarii
- Zapewnienie, że zawsze utuchomiona jest określona liczba *Pod*-ów
- Umożliwienie bezprzerwowych aktualizacji (*rolling updates*)====
#TODO
## Pojęcia
**Klaster**
- zestaw maszyn (fizycznych lub wirtualnych), które działają razem jako jedna jednostka do uruchomienia aplikacji kontenerowych.
**Węzeł**
- maszyna (fizyczna lub wirtualna) w klastrze Kubernetes. Każdy węzeł uruchamia przynajmniej jeden kontener.
**Master Node**
- zarządza klastrem, kontroluje, które kontenery są uruchamiana na jakich węzłach.
**Worker Node**
- węzeł wykonujący aplikacje. Otrzymuje polecenia od *Master Node*-a, aby uruchomić kontenery.
**Pod**
- najmniejsza i najprostrza jednostka w Kubernetes. Zawiera jeden lub więcej kontenerów, które współdzielą zasoby, takie jak sieć i system plików. *Pod* nie może być podzielony na klika węzłów.
## Architektura
**Klaster Kubernetes** składa się z:
- **Master Node** - odpowiada za zarządzanie klastrem, planowanie zadań, monitorowanie stanu oraz podejmowanie decyzji dotyczących wdrażania aplikacji.
- **Worker Nodes** - to maszyny uruchamiające aplikacje w kontenerach
**Komponenty Worker Node**
- **kubelet** - agent, który zarządza *Pod*-ami uruchomionymi na *Worker Node*, komunikuje się z *Master Node*.
- **kube-proxy** - odpowiada za zarządzanie ruchem sieciowym do i z *Pod*-ów w klastrze.
- **Container Runtime** - oprogramowanie, które obsługuje kontenery, np. Docker, containerd czy CRI-O.
**Komponenty Master Node**
- **kube-apiserver** - centralny punkt komunikacji z klastrem. Odbiera żądania od użytkowników i innych komponentów Kubernetes, które korzystają z API
- **etcd** - rozproszona baza danych, w której przechowywane są wszystkie informacje o stanie klastra
- **kube-scheduler** - odpowiada za przydzielanie *Pod*-ów do odpowiednich *Worker Nodes* na podstawie dostępnych zasobów
- **kube-controller-manager** - uruchamia kontrolery odpowiedzialne za utrzymywanie stanu klastra (np. odtwarzanie *Pod*-ów, skalowanie)
- **cloud-controller-manager** - integruje Kubernetes z platformą chmurową (w przypadku środowisk chmurowych)
## Kubernetes vs. Docker Swarm
**Kubernetes**
- automatyzacja zarządzania kontenerami, replikacją, aktualizacją, load balancingiem
- posiada wsparcie do zaawansowanych funkcji, takich jak autoskalowanie i rolling updates
- popularność wynika z bogatego ekosystemu, wsparcia dużych firm i szerokiej adaptacji w świecie *open-source* 

**Docker Swarm**
- prostrzy w konfiguracji, ale oferuje mniejszą elastyczność i funkcjonalność
- działa dobrze dla mniejszych środowisk, ale ma problemy ze skalowaniem na większe produkcje

**Dlaczego Kubernetes?**
- lepsza obsługa dłużych klastrów
- rozbudowane możliwości automatyzacji i zarządzania
- szerokie wsparcie od dostawców chmurowych i duża społeczność