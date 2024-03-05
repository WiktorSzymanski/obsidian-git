##### Lab 1 - cdn.
### SquashFS
Tylko read only, można wykorzystać do archiwizacji. Obraz jest skompresowany, dekompresja ma miejsce przy odczytywaniu poszczególnych plików.
```
mksquashfs dir disk.img -> przeskanowanie i utworzenie obrazu
```
Deduplikacja, widoczna przy np. mailach gdzie dla wielu maili wysyłamy ten sam plik.
```
dd if=/dev/urandom of=some.dir bs=1M count=100
```

### Rozszerzone atrybuty, metadane
Brak jasno okreśnolego ile mogą one zajmować. 
```
setfattr -n user.atr1 -v "something" file.txt -> dodanie metadanych
getfattr -d file.txt -> wyświetlenie metadanych
cp --preserve=xattr file.txt copy.txt -> kopiowanie z zachowaniem metadanych
```

### Atrybuty plików
```
chattr +a file.txt -> dodanie atrybutu a (append only)
lsattr file.txt -> wyświetlanie atrybutów
```
Może się przydać by np. za pomocą i (immutable) nie usunąć przez przypadek czegoś czego nie chcemy.

### NTFS
```
ntfs-3g -s strams_interface=windows disk.img dir -> montowanie ntfs ze strumieniami

echo "hello" > file.txt:inny -> dodanie do strumienia inny "hello"
```
strumienie poboczne się nie kopiują ze zwykłym cp.


### Zmiana rozmiaru systemu plików
Do zwiększenia rozumiaru systemu plików używamy komend specyficznych dla danego systemu plików. Dla ext4 jest to:
```
resize2fs /dev/loop
(chyba przed trzeba zrobić:)
losetup -c /dev/loop1
df -h /mnt -> pokazuje jakie dyski używa dany dir (?)
``` 

```
dd if=/dev/zero of=/mnt/out bs=1M -> alokuje całą pamięć, bo nie ma count=N
```

### Systemy plików w pamięci
zram -> kompresowany system plików w pamięci.
```
modprobe zram -> załadowanie do jądra modułu zram
zramctl -f -a zstd -s 2G -> allokuje nowe urządzenie blokowe z RAM
zramctl -r /dev/zram0 -> usunięcie urządzenia blokowego z pamięci podręcznej
```
Używa pamięć RAM dopiero gdy coś tam jest. Zastosowaniem może być umieszczenie w takim dysky pamięci _swap_. Ze względu na swoją kompresowalność będziemy mieli "więcej" pamięci RAM, minusem jest dodatkowe obciążenie processora.

### Grupowanie systemow plików
```
zypper install unionfs-fuse
```
unionfs usunięcie pliku reprezentje jako plik.txt_HIDDEN~ w .unionfs, jako że nie może usunąć pliku z READ_ONLY.

### LVM
```
pvs -> phisical volume states
pvdisplay -> bardziej szczegółowy raport
vgcreate vg0 /dev/sda3 /dev/sda4 -> tworzy witrualną grupę z fizycznych woluminów
```