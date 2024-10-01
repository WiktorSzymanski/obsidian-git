---
class: KUB2024
---
# Blue/Green
---
> Użytkownicy dostają się do aplikacji przez jakiś punkt wejścia (Load Balancer) do podów *Blue*. W momencie gdy chcemy przetestować jakąś nową wersję, stawiamy pody *Green* z innym punktem wejścia, do testowania. Może być testowana przez ludzi jak i boty. Te dwie wersje egzystują obok siebie aż do momentu podjęcia decyzji wprowadzenia nowych zmian. Wtedy punkt dostępu użytkowników zaczyna wskazywać na pody Green.

##  Kiedy
- duża zmiana
- duże ryzyko
- długie testy (w izolacji)

#### Zalety
- bez *downtime*
- prostota (technicznie)

#### Wady
- wielkie boom! (w mgnieniu oka wszyscy użytkownicy będą łączyć się z nową wersją)
- cache
- zapewnienie x2 więcej zasobów

## Jak wdrożyć
- Service
- Ingress
- Argo Rollouts
- Flagger
- Service Mesh np. Istio (do switcha) 