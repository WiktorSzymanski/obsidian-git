#SRC #Sem1 #ZSK #wykład

- Podział oprogramowania na komponenty
	- Bezpieczeństwo - ograniczone możliwości przy próbie włamania jeśli mamy tylko te pakiety które potrzebujemy 
- Pakiet - pojedyńcze archiwum z wydzielonego komponentu
- Baza danych plików i uprawnień zainstalowanego oprogramowania
- Zewnętrzne repozytoria pakietów

#### Zalety
- Ujednolicona metoda instalacji oprogramowania
- Szybkość instalacji - brak konieczności kompilacji przy instalacji plików binarnych
- Prosta deinstalacja - odwołując się do bazy
- Możliwość automatyzacja przetwarzania (tryb wsadowy)
- Możliwość prostego samodzielnego kompilowania pakietów źródłowych
- Podpisywanie pakietów cyfrowo, daje nam gwarancja że twórca/dystrybutor tego pakietu go skompilował
- Aktualizacja inkrementacja - nie przesłyanie całego programu jeśli zaktualizowany został jeden plik
#### Pakiety binarne
- są binarne - szybsza instalacja, mniejsza wymagana wiedza -> kompilacja środowiska KDE, OpenOffice, MozillaFirefox
- są mniejsze - szybsza instalacja z sieci

#### Samodzielna kompilacja
- strojenie parametrów konfiguracji (docelowy procesor , opcjonalne moduły)
- (teoretycznie) bezpieczniejsza - nie ma nic ponad kod źródłowy, jeśli wiemy że kod źródłowy jest bezpieczny
- możliwość wprowadzenia lokalnych modyfikacji do kodu

#### Granulacja pakietów
- Wydzielanie wspólnych (dynamicznie ładowanych) bibliotek i zewnętrznych narzędzi (`libreadline`, `sqlite3`, `zlib`)
	- oszczędzanie miejsca
	- statyczne dołącznie bibliotek: problemy z aktualizacjami (np. pakiet `zip`)
- Podział na mniejsze pakiety
	- samba -> samba + samba-client + samba-devel + samba-doc + samba-winbind
- Rozbudowanie zależności vs zajętość dysku - mniej miejsca zajmują bo instalujemy tylko to co potrzebujemy
- Aktualizacje muszą uwzględniać zależności
- Poleganie na konkretnej wersji bibliotek/narzędzi

#### Relacje między pakietami
- Wymaganie pakietu (A requires B)
	- konkretnego pakietu
	- konkretnej wersji pakietu
	- wirtualnego pakietu (własności)
	- biblioteki
	- konkretnej wersji biblioteki
	- konkretnego pliku (np. `/bin/sh`)
	- do instalacji i wymagane do kompilacji
- Pakiety wirtualne (C provides B)
	- np. `news_reader -> tin | trn | thunderbird`
	- `ftp_server -> pureftp | vsftpd | proftpd`
- Zalecanie innego pakietu (_A rocommends B_, _A suggests B_)
- Konflikt (_A confilicts with B_)
	- wspólne pliki konfiguracyjne np. serwery NFS i plik `/etc/exports`
- Zastępowanie innego pakietu (np. sdk -> java)


#### Procedura instalacji pakietu
#TODO
#### Procedura usuwania pakietu
#TODO
#### Procedura aktualizacji pakietu
#TODO 

Dlaczego tworzyć pakiety?
- dystrybucja włansego oprogramowania
- brak odpowiedniego pakietu RPM
#TODO