---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 17
---
# 17. Problematyka zarządzania oprogramowaniem – uzupełnienie
---
> Uzupełnienie notatek [[Zarządzanie Systemami Komputerowymi/Pakiety Oprogramowania]] i [[Zarządzanie Systemami Komputerowymi/RPM]]: procedury instalacji, usuwania i aktualizacji pakietu, plik `.spec`, wyzwalacze oraz dystrybucja aktualizacji.

## Problemy zarządzania oprogramowaniem (przypomnienie)
Instalacja, aktualizacja, deinstalacja, lokalizacja plików, weryfikacja (wykrywanie modyfikacji), rozwiązywanie zależności i konfliktów, automatyzacja na wielu maszynach – [[Zarządzanie Systemami Komputerowymi/Zarządzanie oprogramowaniem]].

## Procedura instalacji pakietu (`rpm -i`)
1. **Weryfikacja podpisu** (GPG) i sumy kontrolnej pakietu.
2. **Sprawdzenie zależności** (`Requires`) – czy w bazie RPM są pakiety/biblioteki/pliki udostępniane (`Provides`) w wymaganych wersjach.
3. **Sprawdzenie konfliktów** (`Conflicts`) oraz kolizji plików z innymi pakietami.
4. Uruchomienie skryptu **`%pre`** (np. utworzenie użytkownika systemowego).
5. **Rozpakowanie plików** z archiwum cpio, ustawienie właścicieli, uprawnień i atrybutów.
   - pliki konfiguracyjne (`%config`): jeśli istnieje zmodyfikowany plik → nowy zapisywany jako `.rpmnew` (`%config(noreplace)`) lub stary przenoszony do `.rpmsave`.
6. Uruchomienie skryptu **`%post`** (np. `ldconfig`, rejestracja usługi).
7. **Aktualizacja bazy RPM** (lista plików, sumy MD5/SHA, metadane).
8. Uruchomienie **wyzwalaczy** innych pakietów reagujących na instalację tego pakietu.

## Procedura usuwania (`rpm -e`)
1. Sprawdzenie, czy **inne pakiety nie wymagają** usuwanego (odwrotne zależności) – inaczej odmowa.
2. Skrypt **`%preun`** (np. zatrzymanie usługi).
3. Usunięcie plików z listy pakietu; **zmodyfikowane** pliki konfiguracyjne zachowywane jako `.rpmsave`.
4. Skrypt **`%postun`**.
5. Usunięcie wpisów z bazy; wyzwalacze `%triggerun`/`%triggerpostun`.

## Procedura aktualizacji (`rpm -U`, `-F`)
1. Sprawdzenie zależności nowej wersji (i tego, czy inne pakiety nie wymagają starej wersji).
2. `%pre` **nowego** pakietu.
3. Instalacja plików nowej wersji (obsługa `%config`: porównanie sum oryginalnej, bieżącej i nowej wersji).
4. `%post` **nowego**.
5. `%preun` **starego**.
6. Usunięcie plików starej wersji, **których nie ma** w nowej.
7. `%postun` **starego**.

Skrypty dostają argument `$1` = liczba instancji pakietu po operacji: przy instalacji `%post $1=1`, przy aktualizacji `%post $1=2`, a `%postun` starego dostaje `$1=1`; przy usunięciu `%postun $1=0`. Pozwala to odróżnić aktualizację od usunięcia (np. nie kasować użytkownika przy aktualizacji).
- `-U` instaluje lub aktualizuje; `-F` (_freshen_) aktualizuje tylko zainstalowane.
- Przy wysokopoziomowych narzędziach (`zypper`, `dnf`, `apt`) rozwiązywane są zależności (SAT solver w libsolv), pobierane pakiety z repozytoriów i wykonywana **transakcja** na wielu pakietach.

## Plik `.spec` (receptura budowania RPM)
```spec
Name:           hello
Version:        2.10
Release:        1%{?dist}
Summary:        Program wypisujący powitanie
License:        GPLv3+
URL:            https://www.gnu.org/software/hello/
Source0:        https://ftp.gnu.org/gnu/hello/%{name}-%{version}.tar.gz
Patch0:         hello-fix-typo.patch
BuildRequires:  gcc, make
Requires:       glibc

%description
Program GNU Hello.

%prep                       # przygotowanie źródeł
%setup -q                   # rozpakowanie Source0
%patch0 -p1                 # nałożenie łatki

%build                      # kompilacja
%configure
make %{?_smp_mflags}

%install                    # instalacja do katalogu tymczasowego $RPM_BUILD_ROOT
make install DESTDIR=%{buildroot}

%check                      # testy
make check

%pre
getent passwd hello >/dev/null || useradd -r hello

%post
/sbin/ldconfig

%postun
/sbin/ldconfig

%files                      # lista plików pakietu
%license COPYING
%doc README
%{_bindir}/hello
%config(noreplace) %{_sysconfdir}/hello.conf

%changelog
* Mon Jan 01 2024 Jan Kowalski <jan@example.com> - 2.10-1
- Pierwsze wydanie
```
- **Preambuła** (nagłówek): `Name`, `Version`, `Release`, `Summary`, `License`, `Source`, `Patch`, `BuildRequires`, `Requires`, `Provides`, `Conflicts`, `Obsoletes`, `BuildArch`.
- **Sekcje budowania**: `%prep`, `%build`, `%install`, `%check`, `%clean`.
- **Skryptlety**: `%pre`, `%post`, `%preun`, `%postun`, `%pretrans`, `%posttrans`.
- **`%files`** z dyrektywami `%config`, `%doc`, `%license`, `%dir`, `%attr(mode,user,group)`, `%ghost`.
- Makra: `%{_bindir}`, `%{_sysconfdir}`, `%{buildroot}`, `%{?dist}`.
- Budowanie: `rpmbuild -ba hello.spec` (binarny + źródłowy SRPM), zalecane jako **zwykły użytkownik**, w czystym środowisku (`mock`, OBS).

## Wyzwalacze (_triggers_)
Skrypty pakietu A uruchamiane, gdy **inny** pakiet B jest instalowany lub usuwany:
- `%triggerin -- B` – po instalacji B (lub gdy A instalowany, a B już jest),
- `%triggerun -- B` – przed usunięciem B,
- `%triggerpostun -- B` – po usunięciu B.

Przykład: pakiet serwera WWW rejestruje moduł, gdy zainstalowany zostanie pakiet PHP. Nowsze RPM mają też **file triggers** (`%filetriggerin -- /usr/lib`) – reakcja na instalację plików w katalogu (np. jedno wywołanie `ldconfig` dla całej transakcji). Zob. [[Zarządzanie Systemami Komputerowymi/Wyzwalacze (triggers)]].

## Biblioteki współdzielone – szczegóły
- **soname** (`libgmp.so.3`) zapisany w bibliotece; program linkowany z biblioteką zapamiętuje soname (`NEEDED` w ELF).
- Zmiana **głównego numeru** = zmiana ABI (niekompatybilna) → nowe soname, możliwa **równoległa instalacja** wersji (`libfoo1`, `libfoo2`).
- `ldconfig` aktualizuje dowiązania i cache `/etc/ld.so.cache`; `ldd` pokazuje zależności programu.
- RPM automatycznie generuje `Provides: libgmp.so.3()(64bit)` i `Requires` na podstawie ELF.
- Linkowanie statyczne – brak zależności, ale aktualizacja bezpieczeństwa biblioteki wymaga przebudowy wszystkich programów.
- Szczegóły: [[Zarządzanie Systemami Komputerowymi/Biblioteki]].

## Dystrybucja aktualizacji
- **Repozytoria** z metadanymi (`repodata/repomd.xml`, `primary.xml`) podpisanymi GPG; mirrory; lokalne mirrory/proxy (Spacewalk/Uyuni, Pulp, apt-cacher) – [[Zarządzanie Systemami Komputerowymi/Repozytoria pakietów]].
- **Pakiety różnicowe**: delta RPM (binarna różnica `xdelta` → odtworzenie pełnego pakietu lokalnie), patch RPM (tylko zmienione pliki) – [[Zarządzanie Systemami Komputerowymi/Delta RPM]], [[Zarządzanie Systemami Komputerowymi/Patch RPM]].
- **Łatki bezpieczeństwa**: kanały update, klasyfikacja (security/recommended/optional), `zypper patch`, `dnf update --security`.
- **Automatyzacja**: `unattended-upgrades`, `dnf-automatic`, zarządzanie konfiguracją (Ansible – [[Zarządzanie Systemami Rozproszonymi/Ansible]]), fazowe wdrażanie aktualizacji (testowe → produkcyjne).
- **Aktualizacje atomowe**: systemy obrazowe (OSTree, openSUSE MicroOS z `transactional-update` na migawkach Btrfs) – aktualizacja w nowej migawce, przełączenie przy restarcie, łatwy rollback.
- **Aktualizacje jądra bez restartu**: kpatch, kGraft, livepatch.
- **Formaty samowystarczalne**: AppImage, Flatpak, Snap – aplikacja z zależnościami, niezależna od dystrybucji ([[Zarządzanie Systemami Komputerowymi/AppImage]]); kontenery ([[Docker]]).
