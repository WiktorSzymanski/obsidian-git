---
up: "[[Systemy Plików]]"
class: ZSK
---
#TODO
# B-tree File System
---

>to zaawansowany system plików, zaprojektowany z myślą o współczesnych potrzebach w zakresie zarządzania danymi. Opracowany przez Oracle, dostępny w systemach opartych na jądrze Linux, jego celem jest poprawić integralność danych, łatwość zarządzania nimi i ich skalowalność. 

Btrfs wykonuje pewne operacje w tle, nie blokując przy tym użytkownika. Stąd jednak działanie niektórych operacji zauważymy dopiero po pewnym czasie.
#### Właściwości
- wykorzystuje [[CoW|Copy-on-Write]]
- pozwala tworzyć [[Btrfs#Kopie migawkowe|kopie migawkowe]] na poziomie plików, bez kopiowania całej zawartości plików
- wspiera [[Btrfs#Woluminy wewnętrzne|pod-woluminy]]
- posiada wbudowaną kontrolę błędów i naprawę danych.
- wbudowane [[RAID]] 0, 1, 5, 6 i 10 bez korzystania z zewnętrznych narzędzi. (**Problemy ze stabilnością dla RAID 5 i 6**)
- wspiera transparentną [[Btrfs#Kompresja|kompresję danych]]
- dynamicznie alokuje przestrzeń dyskową

## Woluminy wewnętrzne
---
pozwalają tworzyć _sub_-partycje w partycjach, a następnie je podpinać, a co za tym idzie ukrywać coś w przestrzeni nad podpiętym dyskiem.

## Kopie migawkowe w Btrfs
---
w przeciwieństwie do [[LVM]], który działa pod systemem plików, Btrfs tworzy na poziomie systemu pliku, wykorzystując do tego mechanizm [[CoW]]. Kiedy tworzy [[Kopia Migawkowa|migawkę]], system nie kopiuje danych. Zamiast tego wszelkie nowe modyfikacje zapisuje w nowej lokalizacji, podczas gdy pliki sprzed migawki pozostają w swoim początkowym położeniu.
Dzięki temu, [[Kopia Migawkowa|kopie migawkowe]] w Btrfs, są bardzo szybkie i oszczędne w przestrzeń dyskową.

## Kompresja 
---
Przy szybkim procesorze i wolnym dysku, kompresja może przyśpieszyć operacje wejścia wyjścia. Btrfs podczas kompresji sprawdza czy daje to jakikolwiek zysk, w przeciwnym wypadku nie dokonuje kompresji.