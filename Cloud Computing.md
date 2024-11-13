---
tags:
  - KonstrukcjaSystemówChmurowych
---
# Cloud Computing
---
> **Chmura** jest zwikle kojarzona ze zbiorem wystawionych serwisów, zarządzające zaobami takimi jak:
> - kierowane ekonomią: *pay-as-you-go*
> - skalowalne, avilable on-demand
> - virtualization-based
> - easy to use, internally managed
> - multi-tenant

**Cloud Data Management Interface** to interfejs REST do zarządzania serwisami chmurowymi. Powstał on w celu **ustandaryzowania** serwisów przetwarzania chmurowego.

## Modele wystawiania aplikacji

|                  | CAPEX | OPEX |
| ---------------- | ----- | ---- |
| Own data center  | $$$   | $    |
| Colocation       | $$    | $$   |
| Managed hossting | 0     | $$$  |
| Cloud Computing  | 0     | $$   |

CAPEX- capital expenditute
OPEX - operational expenditute

### Ogolny koszt infrastruktóry
#TODO obrazek z prez
## Cloud computing stack
- [[Software as a Service]]
- [[PaaS|Platform as a Service]]
- [[IaaS|Infrastracture as a Service]]
## XaaS
SaaS + PaaS + IaaS → SPI Model

Storage as a Service
Database as a Service
Framework as a Service
Communication as a Service
Network as a Service
Monitoring as a Service
Desktop as a Service

Anything as a Service (Xaas, EaaS, \*aaS)

## Ogólne problemy
- prywatność danych, bezpieczeństwo:
	- PaaS i SaaS - brak wielu opcji
	- IaaS możliowść szyfrowania zapisywanych danych. Może okazać się to nie efektywne w przypadku gdy dostawca udostępnia nam błędnie działającą infrastrukturę.
#TODO


## Gdzie nie stosować
- Przestarzałe systemy
	- przestarzały *hardware*
	- stare narzędzia deweloperskie, języki programowania
	- wymagają przepisania całej aplikacji
- Systemy *Real-time* z bardzo crytyczne w pewnych scenariuszach - np. jakieś systemy szpitalne
- Przechowywanie poufnych danych

## Gdze stosować
- *Startups* - gdy chcemy sprawdzić jak przyjmie się dany koncept, ale nie chcemy inwestować dużej ilości pieniędzy.
- Małe i średnie biznesy
	- serwisy korporacyjne
	- testowanie nowch produktów/usług
	- kopie zapasowe

## Bezpieczeństwo
- Łagodzi ataki [[DDoS]]
- syfrowanie danych
- [[Paravirtualizacja]]
### Amazon EC2
#TODO