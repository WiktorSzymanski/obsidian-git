# Server OSD
---
- repliki primary / secondary / tertiary
- typowo 1 serwer na 1 dysk (lub macierz)
- Linux + system plików z obsłuhą XATTR (stan, medatane, ACL)
- lokalny RAID nieopłacalny (tylko RAID 0)

## Wybór systemu plików
**Ext4** - 
**XFS** - 
**Btrfs** - wsparcie dla [[CoW]] i modyfikowalnych kopii migawkowych, transparentna kompresja, sumy kontrolne #TODO