---
up: "[[Systemy Plików]]"
tags: ZarządzanieSystemamiKomputerowymi
---
# Księgowanie
---

(ang. _journaling_), polega na cyklicznym zapisywaniu wszystkich wykonanych operacji do logu. Dzięki temu w razie awarii wiemy czy dany plik zapisał się poprawnie (podobnie jak w systemach zarządzania bazami danych), bez konieczności długotrwałego skanowania, by dowiedzieć się czy utraciliśmy jakieś pliki.

Odnosi się do metadanych plików, takich jak np. które bloki zajmuje plik. Ze względu na swoją praktyczność księgowanie stało się standardem.

### Implementacje
---
- JFS
- NTFS
- HFS+ (Apple)
- ext3/4
- XFS
- ReiserFS
- Btrfs