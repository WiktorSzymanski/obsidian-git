---
tags:
  - ZarządzanieSystemamiRozproszonymi
up: "[[Monitorowanie w Systemach Rozproszonych]]"
---
# Grafana
---
>**Grafana** służy do wirtualizacji danych z [[Prometheus]]-a i innych źródeł, umożliwiając tworzenie interaktywnych *dashboard*-ów. Dzięki **Grafana** można śledzić metryki wydajności systemu, analizować trendy i szybko identyfikować problemy. 

Służy tylko do wyświetlania logów Istnieje pod-moduł **Grafana loki** do przeglądania logów.

#### Przykład konfiguracji dashboard-u
Aby skonfigurować **Grafana** do wizualizacji danych z [[Prometheus]] należy:
- Dodać [[Prometheus]] jako źródło danych
- Utworzyć panele przedstawiające kluczowe metryki, np. zużycie CPU, pamięci, ruch sieciowy
- Zdefiniować filtry i wykresy, aby lepiej analizować dane. Można dodać alerty, które będą wyzwalane, gdy określone wartości metryk zostaną przekroczone.