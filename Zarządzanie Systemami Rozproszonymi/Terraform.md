---
up: "[[IaC]]"
tags:
  - ZarządzanieSystemamiRozproszonymi
---
# Terraform
---
> **Terraform** to narzędzie open-source do zarządzania infrastrukturą jako kod stworzone przez HashiCorp. Pozwala tworzyć i zarządzać infrastrukturą na różnych platformach chmurowych, w tym AWS, Azure, Google Cloud. **Terraform** definiuję infrastruktórę za pomocą deklaratywnego języka **HCL** (**HashiCorp Configuration Language**), opisując zasoby. Zarządzanie infrastrukturą jest oparte na stanie (*state*), co pozwala na porównywanie pożądzanego i aktualnego stanu zasobów. **Terraform** po zmianach licencyjnych został "sforkowany" do projektu OpenTofu.

**Terraform** wspiera zasoby chmurowe, takie jak EC2 w AWS, VM w Azure, [[Kubernetes]], bazy danych, load balancery, VPC i więcej. Jedna konfiguracja może być uruchamiana na różnych platformach dzięki dostwcom (*providers*) **Terraform**, np. `provider "aws"`, `provider "google"`.
## Działanie
Planowanie i stosowanie zmian to proces obejmujący **trzy kroki**:
1. Pisanie kodu
2. Planowanie (*terraform plan*)
	- generuje plan pokazujący, jakie zasoby będą utworzone, zmienione lub usunięte
3. Wdrażanie (*terraform apply*)
	- wdraża planowane zmiany, tworząc i konfigurując zasoby.

> Deklaratywne podejście zapewnia, że każdy zasób zostanie zaktualizowany zgodnie z oczekiwaniami.

## Moduły
**Moduły Terraform** to wielokrotnego użytku zestawy plików konfiguracyjnych, umożliwiające tworzenie powtarzalnych elementów infrastruktury. Organizują infrastrukutrę strukturalnie, co ułatwia skalowanie i współpracę. **Moduły** mogą być tworzone samodzielnie lub pobierane z **Terraform Registry** - repozytorium gotowych modułów.