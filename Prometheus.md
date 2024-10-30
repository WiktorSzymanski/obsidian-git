---
tags:
  - ZarządzanieSystemamiRozproszonymi
up: "[[Monitorowanie w Systemach Rozproszonych]]"
---
# Prometheus
---
> **Prometheus** to narzędzie *open-source* do monitorowania i alertowania, które przechowuje dane jako szeregi czasowe (*time-series*). Zbiera metryki z aplikacji i infrastruktóry za pomocą metody *"scraping"*. Jest popularny w środowiskach [[Kubernetes]], ponieważ dobrze integruje się z jego komponentami.

**Prometheus** działa w oparciu o architekturę *pull*, gdzie regularnie pobiera metryki (*scrapping*) z różnych źródeł, takich jak eksportery czy aplikacje. Dane są przechowywane w lokalnej bazie danych jako szeregi czasowe. Użytkownicy mogą definiować regóły alertów, które będą wyzwalane na podstawie określonych wartości metryk. **Prometheus** posiada wsparcie dla eksportowania danych do innych systemów i integracji z [[Grafana]].

## Typy metryk
W **Prometheus** istnieją trzy główne typy metryk:
- Licznik (*Counter*) - metryka monotoniczna, która zawsze rośnie, np. liczba obsłużonych żądań HTTP.
- Histogram - metryka umożliwiająca śledzeie rozkładu wartości takich jak np. czas odpowiedzi serwera dzieląc je na "wiadra"
- Gauge - metryka zmieniająca się w czasie, np. aktualne zużycie pamięci lub obciążenie procesora.

#### Przykład architektury
![[1_2Rxak3lZ3pB5_dx1uXeB8Q.webp]]

## Eksportery
**Eksportery** to programy zbierające metryki z określonych systemów, np. Node Explorer dla metryk systemu operacyjnego. Działa jako pewnego rodzaju *middleware* w procesie przetwarzania logów/metryk.

## Alerting
**Alerting** to system wykrywania problemów na podstawie zebranych metryk, np.:
- wysokie zużycie CPU przez dłuższy czas
- liczba błędów HTTP 5xx przekraczająca ustalony próg
