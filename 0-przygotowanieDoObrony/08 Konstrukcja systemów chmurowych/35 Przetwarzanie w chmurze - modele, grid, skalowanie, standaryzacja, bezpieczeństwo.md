---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 35
---
# 35. Przetwarzanie w chmurze: modele SPI (XaaS), motywacje ekonomiczne, chmura a grid, (auto)skalowanie, standaryzacja, chmury prywatne i hybrydowe, bezpieczeństwo
---
> **Przetwarzanie w chmurze** (NIST SP 800-145) to model umożliwiający wygodny dostęp sieciowy **na żądanie** do **współdzielonej puli konfigurowalnych zasobów** obliczeniowych (sieci, serwery, pamięć masowa, aplikacje, usługi). Zasoby można szybko przydzielić i zwolnić przy minimalnym wysiłku zarządczym i minimalnej interakcji z dostawcą.

Istniejące notatki: [[Konstrukcja Systemów Chmurowych/Cloud Computing]], [[IaaS]], [[Konstrukcja Systemów Chmurowych/PaaS]], [[Konstrukcja Systemów Chmurowych/Software as a Service]], [[Konstrukcja Systemów Chmurowych/Prywatne Chmury]].

## Pięć zasadniczych cech (NIST)
1. **Samoobsługa na żądanie** (_on-demand self-service_) – konsument sam przydziela zasoby bez udziału człowieka po stronie dostawcy.
2. **Szeroki dostęp sieciowy** (_broad network access_) – przez standardowe mechanizmy i różne urządzenia.
3. **Pula zasobów** (_resource pooling_) – zasoby dostawcy współdzielone przez wielu klientów (**multi-tenancy**), przydzielane dynamicznie; klient nie zna dokładnego położenia (co najwyżej region).
4. **Szybka elastyczność** (_rapid elasticity_) – zasoby zwiększane i zmniejszane szybko, często automatycznie; dla klienta sprawiają wrażenie nieograniczonych.
5. **Mierzalność usługi** (_measured service_) – automatyczne pomiary wykorzystania (godziny CPU, GB, żądania) → przejrzystość i rozliczanie za użycie.

## Modele usług – SPI i XaaS
| Model | Klient dostaje | Klient zarządza | Dostawca zarządza | Przykłady |
|---|---|---|---|---|
| **IaaS** | maszyny wirtualne, sieci, dyski | SO, middleware, runtime, aplikacje, dane | wirtualizacja, serwery, pamięć, sieć, centrum danych | AWS EC2, Azure VM, GCE, OpenStack |
| **PaaS** | środowisko uruchomieniowe aplikacji | aplikacje i dane | + SO, runtime, middleware, skalowanie | Heroku, Google App Engine, Azure App Service, Cloud Foundry |
| **SaaS** | gotowa aplikacja | konfiguracja, dane użytkownika | wszystko | Office 365, Gmail, Salesforce |

**SPI** = SaaS + PaaS + IaaS. **XaaS** (_Anything as a Service_) – rozszerzenia: **FaaS/serverless** (funkcje uruchamiane zdarzeniami, rozliczanie per wywołanie i ms – AWS Lambda), **CaaS** (kontenery – EKS, GKE), **DBaaS** (RDS, Cosmos DB), **STaaS** (S3), **DaaS** (pulpit), **NaaS**, **MaaS** (monitoring), **BaaS** (backend dla aplikacji mobilnych), **AIaaS**/MLaaS, **SECaaS**.

## Motywacje ekonomiczne
- **CAPEX → OPEX**: brak inwestycji w sprzęt i serwerownię, płatność za użycie (**pay-as-you-go**) – [[Konstrukcja Systemów Chmurowych/Cloud Computing#Modele wystawiania aplikacji]].
- **Elastyczność zamiast nadmiarowej pojemności**: własna infrastruktura musi być zwymiarowana na **szczyt** obciążenia (sezonowość – [[IaaS#Przykład sezonu z Amazon-em AWS]]), a przez resztę czasu jest niewykorzystana (typowo 10–20% utylizacji serwerów). W chmurze płaci się za rzeczywiste obciążenie. Asymetria kosztów niedoboru (utracone przychody) i nadmiaru.
- **Asocjatywność kosztów**: 1000 maszyn przez 1 godzinę kosztuje tyle co 1 maszyna przez 1000 godzin → szybsze wyniki bez dodatkowego kosztu (obliczenia wsadowe).
- **Efekt skali** dostawcy: hurtowe zakupy sprzętu i energii, automatyzacja (1 administrator na tysiące serwerów), lokalizacja centrów danych (tania energia, chłodzenie), wysoka utylizacja dzięki multipleksacji statystycznej wielu klientów.
- **Szybkość wprowadzania produktów** (_time-to-market_), niski próg dla startupów, eksperymenty bez ryzyka.
- **Przeniesienie ryzyka** (awarie sprzętu, planowanie pojemności) na dostawcę.
- **Modele cenowe**: instancje na żądanie, **zarezerwowane** (zobowiązanie 1–3 lata, rabat do ~70%), **spot/preemptible** (niewykorzystana pojemność, tanio, może zostać odebrana), savings plans; opłaty za transfer **wychodzący** (_egress_).
- **Kiedy nieopłacalna**: stałe, przewidywalne i duże obciążenie (własne może być tańsze), wysokie koszty transferu danych, wymagania regulacyjne; ukryte koszty (egress, licencje, migracja, FinOps).
- **TCO** – całkowity koszt posiadania (sprzęt, energia, ludzie, licencje, powierzchnia) jako podstawa porównania.

## Chmura a grid
**Grid** (Foster, Kesselman, 1998) – federacja zasobów obliczeniowych należących do **wielu niezależnych organizacji** (wirtualne organizacje), udostępniana wspólnie do obliczeń naukowych (EGEE/EGI, Open Science Grid, WLCG dla CERN, BOINC w wersji ochotniczej).

| Kryterium | Grid | Chmura |
|---|---|---|
| Własność zasobów | wiele organizacji (federacja) | jeden dostawca (lub organizacja) |
| Model biznesowy | współdzielenie w ramach wspólnoty, przydziały (granty, kwoty) | komercyjny, pay-as-you-go |
| Główne zastosowanie | obliczenia naukowe HPC, wsadowe zadania | usługi biznesowe, web, dowolne obciążenia, interaktywne |
| Jednostka zasobu | zadanie w kolejce (_batch_) | VM / kontener / usługa na żądanie |
| Dostęp | kolejki zadań (PBS, Condor), oczekiwanie | natychmiastowy, samoobsługowy |
| Wirtualizacja | rzadko (fizyczne węzły) | podstawowa technologia |
| Elastyczność | ograniczona, zasoby stałe | wysoka, automatyczne skalowanie |
| Heterogeniczność | duża (różne systemy, domeny) | ujednolicona w obrębie dostawcy |
| Standardy / middleware | Globus Toolkit (GRAM, GridFTP), OGSA, certyfikaty X.509 (GSI) | API dostawców (de facto), OpenStack |
| Bezpieczeństwo | delegacja zaufania między organizacjami (VOMS, proxy certyfikaty) | IAM dostawcy, multi-tenancy |
| Lokalność danych | pliki przesyłane do obliczeń | dane i obliczenia u tego samego dostawcy |

Wspólne: rozproszenie, współdzielenie zasobów, abstrakcja infrastruktury. Chmura przejęła wiele idei gridu (usługowy dostęp do mocy obliczeniowej – _utility computing_), a gridy coraz częściej używają chmur.

## (Auto)skalowanie
### Pojęcia
- **Skalowanie pionowe** (_scale up_) – większa maszyna (więcej CPU/RAM); ograniczone, zwykle z restartem.
- **Skalowanie poziome** (_scale out_) – więcej instancji za load balancerem; wymaga aplikacji **bezstanowej** lub stanu w zewnętrznym magazynie (sesje w Redis, baza) – [[Systemy Rozproszone Dużej Skali/Big Data#Skalowalność]].
- **Elastyczność** = automatyczne skalowanie w obie strony zgodnie z obciążeniem.

### Strategie autoskalowania
- **Reaktywne progowe** – reguły na metrykach: „CPU > 70% przez 5 min → +2 instancje; CPU < 30% przez 15 min → −1”; **okres wyciszenia** (_cooldown_) przeciw oscylacjom, histereza.
- **Śledzenie celu** (_target tracking_) – utrzymanie metryki na zadanej wartości (np. średnie CPU 50%, 1000 żądań/instancję).
- **Harmonogramowe** – znane wzorce (godziny pracy, Black Friday).
- **Predykcyjne** – model uczenia maszynowego prognozuje obciążenie i skaluje **z wyprzedzeniem** (czas startu instancji!).
- **Na podstawie zdarzeń/kolejek** – długość kolejki komunikatów, liczba zadań ([[KEDA]]); skalowanie do zera (serverless).
- **Metryki**: CPU, pamięć, liczba żądań, opóźnienie (p95), długość kolejki, metryki biznesowe.

### Implementacje
- **AWS Auto Scaling Group**: min/max/desired, szablon uruchomieniowy, health checki, integracja z ELB i CloudWatch.
- **Kubernetes**:
  - **HPA** (_Horizontal Pod Autoscaler_): $desiredReplicas = \left\lceil currentReplicas \times \dfrac{currentMetricValue}{desiredMetricValue} \right\rceil$ (z tolerancją 10% i oknem stabilizacji przy skalowaniu w dół),
  - **VPA** – dostosowanie `requests/limits` podów,
  - **Cluster Autoscaler / Karpenter** – dodawanie/usuwanie **węzłów**, gdy pody nie mieszczą się lub węzły są niewykorzystane,
  - **KEDA** – skalowanie od zdarzeń (Kafka, RabbitMQ, Prometheus), także do zera.
- **Wyzwania**: czas uruchomienia (obrazy, rozgrzewanie JVM – _cold start_), skalowanie warstwy danych (bazy trudniej skalować poziomo – sharding, repliki odczytu), stan sesji, koszty przy błędnych regułach, ochrona przed DDoS generującym koszty – [[Scaling on-demand]].

## Standaryzacja
**Problem**: API i usługi dostawców są własnościowe → **uzależnienie od dostawcy** (_vendor lock-in_), trudna migracja i przenośność, brak interoperacyjności chmur hybrydowych.

| Obszar | Standard | Organizacja |
|---|---|---|
| zarządzanie infrastrukturą IaaS | **OCCI** (_Open Cloud Computing Interface_) – REST API do zasobów obliczeniowych, sieci, pamięci | OGF |
| zarządzanie danymi w chmurze | **CDMI** (_Cloud Data Management Interface_) – REST do kontenerów danych, metadanych, ISO/IEC 17826 | SNIA |
| format maszyn wirtualnych | **OVF** (_Open Virtualization Format_) – pakiet VM (deskryptor XML + dyski), ISO/IEC 17203 – [[Konstrukcja Systemów Chmurowych/Open Virtualization Format]] | DMTF |
| zarządzanie infrastrukturą chmurową | **CIMI** (_Cloud Infrastructure Management Interface_) | DMTF |
| opis topologii i orkestracja aplikacji | **TOSCA** (_Topology and Orchestration Specification for Cloud Applications_) | OASIS |
| kontenery | **OCI** Runtime/Image/Distribution Spec; **CRI**, **CNI**, **CSI** w Kubernetes | Linux Foundation, CNCF |
| definicje i architektura | NIST SP 800-145, SP 500-292 (architektura referencyjna), ISO/IEC 17788/17789 | NIST, ISO/ITU |
| bezpieczeństwo | **CSA CCM** (Cloud Controls Matrix), STAR; ISO/IEC 27017 (kontrole dla chmury), 27018 (dane osobowe) | CSA, ISO |
| przenośność zdarzeń i funkcji | CloudEvents | CNCF |

**Standardy de facto**: API **Amazon S3** (implementowane przez MinIO, Ceph RGW, Wasabi), **Kubernetes** jako warstwa abstrakcji nad chmurami, **Terraform** (wieloplatformowe IaC – [[Zarządzanie Systemami Rozproszonymi/Terraform]]), OpenStack (otwarta implementacja IaaS z API zgodnym częściowo z EC2).

Biblioteki abstrakcji: Apache jclouds, libcloud.

## Modele wdrożenia (NIST)
| Model | Opis | Zalety | Wady |
|---|---|---|---|
| **publiczna** | infrastruktura dostawcy dla ogółu klientów | brak CAPEX, elastyczność, skala | mniejsza kontrola, zgodność z regulacjami, lock-in |
| **prywatna** | infrastruktura dla **jednej organizacji** (własne centrum danych lub hostowana, np. OpenStack, VMware vCloud, Eucalyptus) | kontrola, bezpieczeństwo danych, zgodność, starsze aplikacje | CAPEX, mała skala → mniejsza efektywność, własny zespół |
| **społecznościowa** (_community_) | współdzielona przez organizacje o wspólnych wymaganiach (np. administracja publiczna, uczelnie, medycyna) | podział kosztów, zgodność regulacyjna | złożone zarządzanie |
| **hybrydowa** | połączenie dwóch lub więcej chmur (zwykle prywatnej i publicznej), pozostających odrębnymi bytami, połączonych technologią umożliwiającą przenoszenie danych i aplikacji | elastyczność + kontrola | złożoność, integracja, spójność bezpieczeństwa |

### Chmury hybrydowe – szczegóły
- **Scenariusze**:
  - **cloud bursting** – normalne obciążenie w chmurze prywatnej, szczyty „przelewane” do publicznej,
  - dane wrażliwe lokalnie, przetwarzanie/analizy w publicznej,
  - **disaster recovery** – zapasowe środowisko w chmurze publicznej,
  - migracja stopniowa, dev/test w publicznej, produkcja lokalnie.
- **Wymagania technologiczne**: łącza (VPN IPsec, dedykowane – AWS Direct Connect, Azure ExpressRoute), wspólna tożsamość (federacja AD/Entra ID – [[26 Jednokrotne uwierzytelnianie - SSO]]), spójna sieć (adresacja, DNS), przenośne formaty (kontenery, OVF), zarządzanie i orkestracja wielu środowisk (Kubernetes wieloklastrowy, Azure Arc, Google Anthos, AWS Outposts – sprzęt dostawcy w centrum danych klienta), monitorowanie i polityki bezpieczeństwa obejmujące oba środowiska.
- **Multi-cloud** – korzystanie z wielu chmur publicznych (unikanie lock-in, najlepsze usługi) – nie zawsze połączonych.
- **Virtual Private Cloud (VPC)** – logicznie izolowana sieć klienta **w chmurze publicznej**: własna przestrzeń adresowa (CIDR), podsieci publiczne/prywatne, tablice routingu, bramy internetowe i NAT, **grupy bezpieczeństwa** (stanowe firewalle na instancjach) i sieciowe ACL, połączenie VPN z siedzibą, peering VPC. Daje część cech chmury prywatnej przy zachowaniu ekonomii publicznej – uzupełnienie [[Konstrukcja Systemów Chmurowych/Virtual Private Cloud]].

## Bezpieczeństwo chmury
### Model współdzielonej odpowiedzialności
| Warstwa | IaaS | PaaS | SaaS |
|---|---|---|---|
| dane, klasyfikacja, dostęp użytkowników | klient | klient | klient |
| aplikacja | klient | klient | dostawca |
| runtime, middleware | klient | dostawca | dostawca |
| system operacyjny, łatki | klient | dostawca | dostawca |
| wirtualizacja, serwery, sieć fizyczna | dostawca | dostawca | dostawca |
| centrum danych (bezpieczeństwo fizyczne) | dostawca | dostawca | dostawca |

„Dostawca odpowiada za bezpieczeństwo **chmury**, klient – za bezpieczeństwo **w chmurze**”. Najwięcej incydentów wynika z **błędnej konfiguracji po stronie klienta** (publiczne kubełki S3, otwarte bazy, nadmierne uprawnienia IAM).

### Zagrożenia specyficzne
- **Multi-tenancy** – współdzielenie sprzętu z obcymi: ucieczka z VM/kontenera (_hypervisor escape_), kanały boczne (Spectre/Meltdown, cache timing, _noisy neighbour_), niedostateczna izolacja sieci i danych (resztki danych na dyskach),
- **utrata kontroli nad danymi** – lokalizacja danych i **jurysdykcja** (RODO/GDPR, CLOUD Act), dostęp personelu dostawcy, usuwanie danych po zakończeniu umowy,
- **przejęcie kont i kluczy API** (wyciek kluczy w repozytoriach kodu), niebezpieczne interfejsy i API,
- **błędna konfiguracja** i nadmierne uprawnienia,
- **złośliwi insiderzy** (dostawcy i klienta),
- **DoS** i ataki na koszty (_Economic Denial of Sustainability_ – wymuszone autoskalowanie),
- **uzależnienie** i dostępność dostawcy (awarie regionów), upadłość dostawcy,
- **zgodność** z regulacjami i audyt (brak fizycznego dostępu).

### Mechanizmy ochrony
- **IAM**: najmniejsze uprawnienia, role zamiast kluczy statycznych, MFA dla kont uprzywilejowanych, federacja tożsamości – [[27 Mechanizmy egzekwowania polityki kontroli dostępu]],
- **szyfrowanie**:
  - **w tranzycie** (TLS),
  - **w spoczynku** (dyski, S3 – SSE),
  - klucze zarządzane przez **KMS/HSM**, **BYOK/HYOK** (klucze klienta),
  - **confidential computing** – szyfrowanie **w użyciu** (enklawy Intel SGX/TDX, AMD SEV-SNP),
- **izolacja sieci**: VPC, grupy bezpieczeństwa, prywatne endpointy, segmentacja, WAF, ochrona DDoS (AWS Shield, Cloudflare) – chmura dzięki skali łagodzi ataki DDoS,
- **izolacja obliczeń**: nadzorcy z mniejszą TCB (Nitro, Firecracker), parawirtualizacja/dedykowane hosty,
- **monitorowanie i audyt**: logi API (CloudTrail), SIEM, CSPM (_Cloud Security Posture Management_ – wykrywanie błędnych konfiguracji), CWPP,
- **zgodność i certyfikacje**: ISO 27001/27017/27018, SOC 2, PCI DSS, CSA STAR, C5; SLA i umowy o przetwarzaniu danych,
- **ciągłość działania**: kopie w innych regionach, architektura wielostrefowa (_multi-AZ_),
- **DevSecOps**: skanowanie IaC i obrazów, zarządzanie sekretami (Vault, Secrets Manager).

## Gdzie stosować / nie stosować
Zob. [[Konstrukcja Systemów Chmurowych/Cloud Computing#Gdzie stosować]] i [[Konstrukcja Systemów Chmurowych/Cloud Computing#Gdzie nie stosować]].
