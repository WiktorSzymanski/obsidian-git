---
tags:
  - ZarządzanieSystemamiRozproszonymi
---
# Infrastracture as Code
---
> **Infrastructure as Code** (**IaC**) to podejście, które pozwala zarządzać infrastrukturą za pomocą kodu, eliminując ręczne zarządzanie konfiguracją. Umożliwia **zapisywanie**, **wersjonowanie** i **współdzielenie** infrastruktury podobnie jak kodu aplikacyjnego. **IaC** pozwala na przywracanie infrastruktury do spójnego stanu oraz wdrażanie środowisk w sposób powtarzalny i audytowalny. Deklaratywne podejście **IaC** określa *"co"* chcemy osiągnąć, a nie *"jak"* – narzędzie wykonuje wszystkie potrzebne kroki.

## Korzyści
- **Automatyzacja** i **eliminacja błędów ludzkich**: Automatyzacja redukuje błędy wynikające z ręcznejkonfiguracji.
- **Spójność środowisk**: **IaC** gwarantuje identyczność środowisk – od deweloperskiego po produkcyjne.
- **Audytowalność** i **zgodność**: Kod IaC jest wersjonowany, co ułatwia audytowanie zmian i zgodność z politykami bezpieczeństwa.
- **Szybkie przywracanie** i **odtwarzanie**: Można przywrócić środowisko do ostatniej stabilnej wersji z repozytorium. To znacząco poprawia [[RTO]].
- **Łatwiejsze skalowanie**: Automatyczne skalowanie zasobów dostosowuje infrastrukturę do zmieniających się potrzeb.

**RTO** - Recovery Time Objective - ile czasu od awarii aby znów działało
**RPO** - - najstarsza kopia #TODO