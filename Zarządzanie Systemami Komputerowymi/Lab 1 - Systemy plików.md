login: zsk
hasło: 24zeteska
### Mechanizm loop  
  w unix partycje to pliki specjalne.  
    sda2 -> a bo pierwszy dysk, drugi miałby b itd.  
              -> 2 bo druga partycja  
* Dodatkowy kurs o systemach unix na stronie prowadzącego w zakładce SOP  
```
lsblk : wyświetla dyski i ich parcycje  
fdisk -l /dev/sda : informacje o danychm dysku  
df -h . : wyświetla ilość wolnego miejsca  
```

loop pozwala utworzyć nowy system pilków z jakiegoś dużego pliku  
```
fallocate -l 1G big.file : szybka alokacja pamięci  
```  
W unix są dostępne pliki specjalne /dev/loop do skojarzenia pliku z urządzeniem blokowym.  
  
mkfs : "make filesystem" - zakładanie nowego systemu plików np. mkfs.ext4 /dev/loop0  
  
W unix dyski muszą być połączone z root-em aby można z nich korzystać. Można to osiągnąć np. przez:  
  mount /dev/loop0 /mnt   
  
Katalog lost+found - trafiają do niego "zagubione" rzeczy, np. nie zapisane z powodu awarii.  
  
umount : odmontowanie dysku od root-a  
loosetup : łączenie/odłączenie pliku z urządzeniem blokowym  
hdparam -t /dev/sda : test odczytu pamięci, bezpieczny dla danych na dysku  
findmnt /mnt : informacje o danym systemie plików  
  
Mechanizm Fuze - users file system  
  
  ![[FUSE_example.png]]
  
Pozwala aby system plików był procesem. Zmiejsza wydajność ale nie powoduje błędów w jąrze co mogło by powodować awarię systemu.  
  
ntfs-3G big.file Win : mount ntfs bez użycia komendy mount  
fusermount -u Win : umount przez fuse  
  
Komendy wyżej może wykonać zwykły user, przy założeniu że jest ustawiona zmienna w configu fuse allow_others.  
  
Pliki konfiguracyjne znajdują się w folderze /etc