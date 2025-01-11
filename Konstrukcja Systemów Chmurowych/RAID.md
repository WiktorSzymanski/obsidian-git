---
tags:
  - KonstrukcjaSystemówChmurowych
---
# Redundant Array of Independent/Inexpensive Disks
---
## RAID 0
#TODO Obrazek z prez
- pojemność n x S
- n krotny wzrost wydajności zapisu i odczytu
- redukcja odporności na awarię - wystarczy że jeden dysk się popsuje i tracimy całe dane

**JBOD** (*just a bunch of disks*) - Dyski połączone ze sobą liniowo, zapis się dzieje pokolej aż do zapełnienia poprzednika. Nie poprawia wydajności zapisu i odczytu.

## RAID 1 / 1E
#TODO 
#TODO obrazek z prez
- wariant 1E pozwala na nieparzystą liczbę dysków

## RAID 2 / 3 / 4
#TODO obrazek z przez
- jeden dysk przechowuje sumy kontrolne
- pojemność (n - 1) x S
- odpornośc na awarię jednego dysku
- efektywność jak **RAID 0**
- kosztowna odbudowa macierzy
- **RAID 4** używa większych bloków (16 : 128KiB vs 1B vs 1bit)
- **RAID 4** mały odczyt angażuje tylko jeden dysk (*RAID 2/3 lockstep - wymuszona synchronizacja dla niewiekich zapisów*)
- problem **RAID 4** z małymi zapisami (*write amplification*)
- mało popularne, zastąpione przez **RAID 5**

## RAID 5 
#TODO obrazek z przez
- sumy kontrolne rozproszone po dyskach
- pojemność (n - 1) x S
- możliwość odczytu z każdego z dysków
- odporność na awarię jednego dysku
- n-krotny wzrost wydajności odczytu
- (n - 1)-krotny wzrost wydajności zapisu
- kosztowna odbudowa macierzy

## RAID 6
#TODO obrazek z przez
- pojemność (n - 2) x S
- posiada dwie sumy kontrolne rozłożone na wszystkich dyskach
- odporność na awarie dwóch dysków
- n-krotny wzrost wydajności odczytu
- (n - 2)-krotny wzrost wydajności zapisu

## Po co nam RAID 6
- awarie dysków: kompletna lub bad block (jest to częstrzy przypadek)
- MTBF dla dysków HDD: 0,5-1,5 mln godzin (50-150 lat)
- rzeczywiste (nieoptymalne) warunk: 5-10 lat
- 5 dysków w macierzy → awaria raz na 1-2 lat
- prawdopodobieństwo jednoczesnej awarii dwóch dysków?
	- raz na 100-10000 lat
- błędy ECC - Error Bit Rate
	- dyski SCSI/SAS: raz na 10^15-10^16 bitów (100TIB - 1PB)
	- dyski SATA: raz na 10^14-10^15 bitów (10TiB - 100TiB)
- dysk SATA 1TIB: 1 na 10 odczytów całości generuje błąd
- odbudowa macierzy składającej się z 5 dysków:
	- SAS 500GB: 10^15/8/5/500/10^9 - raz na 50
	- SATA 1TiB: 10^14/8/5/10^12 - raz na 2,5
	- Na te okoliczność wykorzystuje się **RAID 6**
- **Patrol Read** - sposób minimalizacji ryzyka błędu **ECC**

## Software RAID
- tańszy (kosztowne kontrolery **RAID**)
- obciążanie głównego procesora
- oddzielna partycja startowa
- standard zapisu danych na dyskach
	- łatwiejsze odtwarzanie danych
- możliwość łączania różnych interfejsów (**SCSI**/**SATA**/**USB**)

## Nested RAID
### Zalety
- odbudowa dotyczy tylko części macierzy (np. 1+0 vs 0+1)
- możliwość wykoszystania dysków o różnych pojemnościach i parametrach
- możliwość rozbudowy konfiguracji
- wykorzystanie *software RAID* to łączenia macierzy 
#TODO Obrazek z prez


## Dyski Hot Spare i RAID 5E/6E
- Hot Spare - zapasowy nieużywany dysk na wypadek awarii. W jej przypadku automatycznie wskauje w miejsce tego który uległ awarii
- może być współdzielony dla kilku logicznych macierzy

## RAID 5E
- rozproszony dysk **Hot Spare**
- (n + 1)-krotny wzrost wydajności odczytu
- n-krotny wzrost wydajności zapisu 

## Po co nam RAID
 Dyski twarde są coraz bardziej niezawodne i coraz większe. Z tego faktu ddbudowanie macierzy może trwać ponad dobę, a w tym czasie ryzykujemy kolejną awarię. Konieczność wykorzystywania identycznych dysków jest uciążliwe wraz z kosztownymi kontrolerami RAID. RAID zabezpiecza przed awarią dysku ale nie sieci...

Stąd pomysł na [[ClusterFS]]