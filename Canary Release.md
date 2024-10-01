---
class: KUB2024
---
# Canary Release
---
> Polega na wypuszczeniu kanarka (jakiejś małej części podów w nowej wersji). Następnie jakiś procent (np. 5%) ruchu jest przekierowywany na "kanarka".

W momencie gdy już wiemy, że chcemy wprowadzić zmiany na wszystkie pody możemy użyć do tego [[Rolling update]].

## Kiedy
- kosztowny rollback
- decyzja techniczna
- testy bez dalszego wdrożenia
- testy anonimowe (nie wiemy jacy użytownicy, wiemy jaki procent ruchu)

#### Zalety
- bez *downtime*
- prosto (2/5)

#### Wady
- kompatybilność wsteczna (? no ale niby wystarczy cofnąć kanarka i git)
- ocena wyników -> sposób badania

## Jak wdrożyć
- Ingress (nginx)
- Argo Rollouts
- Flagger
- Service Mesh np. Istio

W praktyce mamy dwie konfiguracje ingressa gdzie jedna z nich  posiada dodatkowe adnotacje nginx, gdzie podajemy procent ruchu jaki ma być przekazywany do "kanarka".