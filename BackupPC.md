#SRC #Sem1 #ZSK

- Metadane są składowane niezależnie od zawartości plików
- Optymalizacja zajętości dysku:
	- deduplikacja
	- szczególnie opłacalne #TODO 

- Brak oprogramowania klienta
	- Windows: dostęp sieciowy protokołem SMB/CIFS
	- Unix, MAC OS X: NFS, SSH (tar) #TODO 

#### Technika
- Wymagana obsługa dowiązań twardych na nośniku archiwizacyjnym
- Bardzo duża liczba plików: dostępna liczba i-węzłów (`df -i`)
- Funkcja haszująca bazę danych plików:
	- MD5 z rozmiaru pliku oraz z początkowych i końcowych 128KiB pliku
	- kompromis pomiędzy czasem obliczeń sumy kontrolnej a czasem porównania zawartości plików z nieunikalnym sktórem

#TODO 