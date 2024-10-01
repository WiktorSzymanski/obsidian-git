---
class: KUB2024
---
# A/B testing
---
> Podobne do [[Canary Release]], ale tutaj nie anonimowo, a świadomie podjemowana jest decyzja kto do jakiej grupy trafia

## Kiedy
- testy UI
- badanie zachowania użytkowników

#### Zalety
- bez *downtime*
- prosto (3/5)

#### Wady
- kompatybilność wsteczna (opcjonalnie)
- implementacja dzielenia grup

## Jak wdrożyć
- Ingress (nginx)
- Argo Rollouts
- Flagger
- Service Mesh np. Istio

Tak jak w wypadku [[Canary Release]], ale tu stosujemy inne adnotacje. To do jakiej wersji kto trafia zależy od nagłówka, który może być mu nadawany np. po zalogowaniu się.