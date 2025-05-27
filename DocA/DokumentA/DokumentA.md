![[PP_znak_konturowy_RGB.png|#center|300]]
</br>
</br>
# Politechnika Poznańska
## Informatyka II stopień
### Systemy Rozproszone i Chmurowe
#### Zarządzanie Bezpieczeństwem w systemach IT
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>
</br>

##### Wiktor Szymański 148084

# 1. Charakterystyka Ogólna
Firma BonkerSolutions (**BS**) to dynamiczny, nowoczesny startup zatrudniający 30 pracowników i zajmujący się tworzeniem systemu rezerwacji różnorodnych przestrzeni użytkowych BooMe. Oferowane przez nas rozwiązanie umożliwia klientom łatwe i intuicyjne rezerwowanie miejsc, takich jak stoliki w restauracjach, tory bowling-owe, czy sale konferencyjne. System został zaprojektowany z myślą o skalowalności dzięki architekturze chmurowej.

# 2. Struktura Organizacyjna
Organizacja składa się z pięciu zespołów, każdy z zespołów z wyłączeniem BigB składa się z 7 pracowników, gdzie jeden z nich pełni rolę managera tego zespołu.
- CoreB
- FrontE
- SaaSy
- DevOn
- BigB

![[TeamImg.png]]

## CoreB
Zespół odpowiedzialny za rozwój systemu, przetwarzanie danych, zarządzanie bazą danych oraz logikę aplikacji. Składa się z wysoko wykwalifikowanych programistów backend-owych.
## FrontE
Zespół skupiony na tworzeniu atrakcyjnej i intuicyjnej oprawy graficznej oraz zapewnianiu pozytywnych doświadczeń użytkowników końcowych (UX/UI). 
## SaaSy
Zespół rozwijający funkcjonalność SaaS, pozwalającą klientom biznesowym na dokonywanie zmian w ustawieniach i konfiguracji swojego systemu bez potrzeby angażowania zespołu technicznego.
## DevOn
Zespół zajmujący się wdrożeniami, automatyzacją procesów, zarządzaniem infrastrukturą i zapewnieniem ciągłości działania systemu. Dwóch pracowników z tego zespołu pełni dodatkowo rolę lokalnych administratorów systemu IT w firmie.
## BigB
Zarząd składający się z właściciela firmy oraz jego zastępcy, odpowiedzialnych za wyznaczanie strategii rozwoju, nadzorowanie kluczowych działań, zarządzanie zasobami ludzkimi oraz podejmowanie decyzji na poziomie biznesowym.

# 3. Plany budynku i lokalizacja

![[PlanBudynku.png]]

Biuro firmy BonkerSolutions znajduje się na Alejach Solidarności, 61-623 Poznań. Główne drzwi do bura znajdują się w pomieszczeniu nr. 6. Aby wejść do biura konieczna jest karta pracownika. Każdy z zespołów posiada swój osobny pokój. Są to pokoje 5, 4, 3, 12 i 13. Aby wejść do takiego pomieszczenia również jest potrzebna karta pracownicza. Na co dzień wejścia do pokoi zespołów nie są blokowane, ma to miejsce w przypadku gdy jest do firmy zapraszany jakiś gość czy potencjalny klient. Pomieszczenie nr. 2 to sala konferencyjna, a nr. 14 to stołówka/chill zone.
# 4. Inwentaryzacja Zasobów IT
## 4.1. Sprzęt

| sprzęt                                          | ilość | specyfikacja                                                                                                                                                |
| ----------------------------------------------- | ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Laptop Dell XPS 15                              | 30    | - Procesor Intel® Core™ i7-1260P (12 rdzeni, 16 wątków, 3.40-4.70 GHz, 18MB cache)<br>- Pamięć RAM 16 GB (LPDDR5x, 5200 MHz)<br>- Dysk SSD M.2 PCIe 1000 GB |
| Monitor GIGABYTE M34WQ                        | 30    | ---                                                                                                                                                         |
| Klawiatura - Nuphy Air96 V2                     | 30    | ---                                                                                                                                                         |
| Mysz - Logi MX Vertical                         | 30    | ---                                                                                                                                                         |
| Headset - EPOS IMPACT 860 ANC Stereo PC Headset | 30    | ---                                                                                                                                                         |
| Cisco Meraki MX75                               | 1     | - Zalecana ilość urządzeń: do 200<br>- Maksymalna przepustowość ze wszystkimi zaawansowanymi zabezpieczeniami: 1 Gbps                                       |
| CANON i-SENSYS MF655Cdw                         | 1     | - WiFi                                                                                                                                                      |
## 4.2. Oprogramowanie

| nazwa                  | ilość |
| ---------------------- | ----- |
| Ubuntu 24.04 LTS       | 30    |
| JetBrains Ultimate     | 30    |
| YouTrack               | 30    |
| Bitwarden              | 30    |
| Bitdefender            | 30    |
| Microsoft Azure Cloud  | -     |
| Microsoft Dynamics 365 | 30    |
| Microsoft Intune       | 30    |
| AWS                    | -     |
| GCP                    | -     |
| Microsoft Entra        | 30    |

Microsoft Dynamics 365 jest używany jako ERP. Bitwarden jest wykorzystywany do przechowywania haseł i kluczy potrzebnych podczas tworzenia oprogramowania. Bitdefender dba o bezpieczeństwo urządzeń pracowników i wykrywanie ewentualnych wirusów. Microsoft Intune pozwala centralnie weryfikować czy zainstalowane oprogramowanie i jego wersja jest zgodna z zasadami i wymogami firmy. Microsoft Entra zapewnia zarządzanie tożsamością i dostępem do zasobów firmy.
# 4. Schematy sieci
Ze względu na nie dużą ilość stanowisk komputerowych i ich stosunkowo ciasne rozmieszczenie, wystarczająca będzie sieć WiFi.
![[TopCor.png]]
Router jest podwieszony na środku korytarza, zabezpieczony w "puszce" przed fizyczną ingerencją.

Sieć pracownicza posiada skonfigurowany VPN i Firewall. Można się do niej dostać tylko z urządzenia o określonym adresie MAC, znajdującym się w puli urządzeń firmowych. Dla gości i prywatnych urządzeń jest skonfigurowana osoba sieć, nie mająca dostępu do firmowych zasobów.
# 5. Dane
Dane klientów i pracowników są przechowywane w systemie ERP, a ich zakodowany backup znajduje się u innego dostawcy chmurowego (AWS lub GCP).

Poza tym jedyne dane jakie przechowuje firma to repozytoria z produkowanym kodem oraz swoje dane testowe. Dane klientów związane z pracą sprzedawanego oprogramowania są przechowywane w chmurze kont klienckich obok wystawionej dla nich aplikacji.