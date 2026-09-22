---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 27
---
# 27. Mechanizmy egzekwowania polityki kontroli dostępu
---
> **Kontrola dostępu** (autoryzacja) decyduje, czy **podmiot** (_subject_ – użytkownik, proces) może wykonać **operację** (odczyt, zapis, wykonanie) na **obiekcie** (_object_ – plik, rekord, zasób sieciowy). **Polityka** określa reguły, a **mechanizm** je egzekwuje. Uwierzytelnianie ([[25 Mechanizmy egzekwowania polityki uwierzytelniania]]) poprzedza autoryzację.

## Zasady projektowe
- **Monitor odwołań** (_reference monitor_, Anderson 1972) – abstrakcyjny komponent, przez który przechodzi **każde** odwołanie podmiotu do obiektu. Wymagania:
  - **pełne pośrednictwo** (_complete mediation_) – każde odwołanie sprawdzane, nie da się go ominąć,
  - **odporność na manipulację** (_tamperproof_),
  - **weryfikowalność** – na tyle mały i prosty, by dało się go przeanalizować.
  Implementacja monitora to **TCB** (_Trusted Computing Base_), np. jądro SO z LSM.
- **Najmniejsze uprawnienia** (_least privilege_) – tylko uprawnienia niezbędne do zadania.
- **Rozdzielenie obowiązków** (_separation of duties_, SoD) – krytyczne operacje wymagają kilku osób/ról.
- **Domyślna odmowa** (_fail-safe defaults_, _default deny_).
- **Obrona w głąb** – wiele warstw kontroli (sieć, system, aplikacja, dane).
- W architekturze polityk (XACML/NIST): **PEP** (punkt egzekwowania) – przechwytuje żądanie; **PDP** (punkt decyzji) – ocenia politykę; **PIP** (punkt informacji) – dostarcza atrybuty; **PAP** (punkt administracji) – zarządza politykami.

## Macierz dostępu (Lampson)
|  | plik1 | plik2 | drukarka |
|---|---|---|---|
| **Alicja** | own, r, w | r | print |
| **Bob** | r | r, w | – |
| **proces P** | – | r | print |

Macierz jest rzadka, więc przechowuje się ją jako:
- **ACL** (listy kontroli dostępu) – **kolumny** przy obiektach: „kto ma jakie prawa do tego obiektu”.
  - ✔ łatwo sprawdzić/odebrać prawa do obiektu,
  - ✘ trudno sprawdzić wszystkie prawa podmiotu,
  - przykłady: uprawnienia UNIX, POSIX ACL, NTFS DACL, AFS.
- **Listy uprawnień / capabilities** – **wiersze** przy podmiotach: nieprzekazywalne/niepodrabialne „bilety” do obiektów.
  - ✔ łatwe delegowanie, brak sprawdzania tożsamości przy każdym dostępie (deskryptory plików, klucze URL, tokeny OAuth, Kerberos tickets),
  - ✘ trudne odwołanie (wszystkie kopie), przeglądanie praw do obiektu.
- **Reguły / polityki** – funkcja decyzyjna zamiast jawnej macierzy (ABAC, firewalle).

## Modele kontroli dostępu
### DAC – uznaniowa kontrola dostępu (_Discretionary_)
- **Właściciel** obiektu sam decyduje o uprawnieniach i może je **przekazywać** innym.
- Przykłady: UNIX `rwx` (właściciel/grupa/inni) + `chmod`/`chown`, POSIX ACL (`setfacl -m u:jan:rw plik`), NTFS ACL.
- ✔ elastyczny, prosty; ✘ podatny na **konie trojańskie** (program uruchomiony przez użytkownika działa z jego prawami i może skopiować dane komuś innemu), brak centralnej kontroli przepływu informacji.

### MAC – obowiązkowa kontrola dostępu (_Mandatory_)
- Polityka narzucona **centralnie** przez system/administratora; użytkownik (nawet właściciel) **nie może** jej zmienić.
- Podmioty i obiekty mają **etykiety bezpieczeństwa** (poziomy tajności + kategorie); decyzja na podstawie porównania etykiet.

**Model Bella-LaPaduli** (1973, **poufność**):
- **prosta własność bezpieczeństwa** (_no read up_) – podmiot może czytać obiekt tylko, jeśli jego poziom ≥ poziom obiektu,
- **własność gwiazdki \*** (_no write down_) – podmiot może zapisywać obiekt tylko, jeśli poziom obiektu ≥ poziom podmiotu (zapobiega „przeciekowi” do niższego poziomu),
- ✘ nie chroni integralności.

**Model Biby** (1977, **integralność**) – dualny:
- **no read down** – nie czytaj danych o niższej integralności,
- **no write up** – nie zapisuj do obiektów o wyższej integralności.

Inne: **Clark-Wilson** (integralność komercyjna: dobrze zdefiniowane transakcje TP, podmiot–program–obiekt, SoD), **Chiński mur / Brewer-Nash** (konflikt interesów – dynamiczne ograniczenia).

Implementacje MAC: SELinux (MLS/MCS), AppArmor, Smack, TrustedBSD, Windows Mandatory Integrity Control, systemy wojskowe.

### RBAC – kontrola oparta na rolach (_Role-Based_, NIST, Sandhu 1996)
- Uprawnienia przypisywane **rolom** (np. „księgowy”, „administrator bazy”), a **użytkownicy** są przypisywani do ról. Użytkownik aktywuje role w **sesji**.
- Poziomy modelu NIST:
  - **RBAC0** (podstawowy): użytkownicy, role, uprawnienia, sesje,
  - **RBAC1** – **hierarchia ról** (dziedziczenie: kierownik ⊇ pracownik),
  - **RBAC2** – **ograniczenia**: statyczne SoD (nie można mieć jednocześnie ról „zlecający płatność” i „zatwierdzający”), dynamiczne SoD (nie aktywować obu w jednej sesji), liczność,
  - **RBAC3** = RBAC1 + RBAC2.
- ✔ odzwierciedla strukturę organizacji, łatwe zarządzanie przy wielu użytkownikach, audyt; ✘ „eksplozja ról”, słabe dla decyzji kontekstowych.
- Przykłady: Kubernetes RBAC (`Role`, `ClusterRole`, `RoleBinding`), grupy AD, role w bazach danych (`GRANT rola TO user`), AWS IAM roles.

### ABAC – kontrola oparta na atrybutach (_Attribute-Based_)
- Decyzja to funkcja **atrybutów**: podmiotu (dział, stanowisko, poziom), obiektu (właściciel, klasyfikacja), **operacji** i **środowiska** (czas, lokalizacja, poziom ryzyka, urządzenie).
- Reguła np.: „lekarz może czytać kartę pacjenta, jeśli pacjent jest przypisany do jego oddziału i jest godzina dyżuru”.
- Standard **XACML** (OASIS) – polityki XML, architektura PEP/PDP/PIP/PAP; nowocześnie: **OPA/Rego**, AWS IAM policies z warunkami, Cedar.
- ✔ bardzo elastyczny, kontekstowy, skalowalny bez mnożenia ról; ✘ trudny do audytu i testowania polityk.

### Inne
- **IBAC** – na podstawie **tożsamości** (listy konkretnych użytkowników) – ACL w czystej postaci,
- **RSBAC** (_Rule Set Based Access Control_) – szkielet dla Linuksa implementujący wiele modułów polityk jednocześnie (MAC, RC – role compatibility, ACL, PM – privacy model),
- **ReBAC** – na podstawie relacji w grafie (Google Zanzibar: „użytkownik jest członkiem grupy, która jest edytorem folderu”),
- **kontrola przepływu informacji** (_information flow control_), **bazująca na historii** (Chiński mur).

Porównanie DAC/MAC w kontekście AppArmor: [[Bezpieczeństwo Systemów Rozproszonych/AppArmor]].

## Mechanizmy w systemach operacyjnych (Linux)
### DAC i uprzywilejowanie
- UID/GID, bity `rwx`, **setuid/setgid** (program wykonywany z prawami właściciela pliku), **sticky bit** (`/tmp`), `umask`, POSIX ACL,
- **sudo** (`/etc/sudoers` – kto, jakie polecenia, jako kto), `su`, polkit,
- **capabilities** – podział uprawnień roota na ~40 niezależnych: `CAP_NET_BIND_SERVICE` (porty < 1024), `CAP_NET_RAW` (gniazda surowe – [[59 Nieprzetworzone gniazda sieciowe - uzupełnienie]]), `CAP_NET_ADMIN`, `CAP_SYS_ADMIN`, `CAP_DAC_OVERRIDE`, `CAP_SYS_PTRACE`; nadawane plikom (`setcap cap_net_raw+ep /usr/bin/ping`) i procesom; zbiory permitted/effective/inheritable/bounding/ambient.

### LSM – Linux Security Modules
Haki w jądrze wywoływane przy operacjach (otwarcie pliku, `exec`, gniazdo) – monitor odwołań dla modułów MAC.
- **SELinux** (NSA, Red Hat):
  - każdy proces i obiekt ma **kontekst** `użytkownik:rola:typ:poziom` (np. `system_u:system_r:httpd_t:s0`),
  - **Type Enforcement** – reguły `allow httpd_t httpd_sys_content_t:file { read open getattr };`, wszystko inne zabronione,
  - **przejścia domen** przy `exec` (`type_transition`), **booleany** (`setsebool httpd_can_network_connect on`),
  - **RBAC** (role → dozwolone typy) i **MLS/MCS** (poziomy – Bell-LaPadula, kategorie – izolacja kontenerów/VM w sVirt),
  - tryby **enforcing** / **permissive** (tylko logowanie AVC denials) / disabled; `audit2allow`, `restorecon`, etykiety w xattr.
- **AppArmor** (SUSE, Ubuntu): **profile oparte na ścieżkach** dla konkretnych programów (`/usr/sbin/nginx { /var/www/** r, network inet tcp, capability net_bind_service, }`), tryby enforce/complain, change hat – [[Bezpieczeństwo Systemów Rozproszonych/AppArmor]].
  - SELinux: etykiety (bezpieczniejsze przy twardych dowiązaniach/zmianie nazw), pełny system, trudniejszy,
  - AppArmor: prostszy, ścieżki, ochrona wybranych aplikacji.
- **Smack**, **TOMOYO**, **Landlock** (piaskownica nieuprzywilejowana), **Yama** (ograniczenie `ptrace`).

### Izolacja i piaskownice
- **seccomp-bpf** – filtr dozwolonych wywołań systemowych (Docker, Chrome, systemd `SystemCallFilter=`),
- **namespaces i cgroups** – izolacja widoku zasobów i limity ([[Konstrukcja Systemów Chmurowych/Konteneryzacja#Namespaces]]),
- `chroot`, jails, maszyny wirtualne.

### Windows
Tokeny dostępu (SID użytkownika i grup, uprawnienia), **deskryptory bezpieczeństwa**: **DACL** (ACE allow/deny), **SACL** (audyt), **poziomy integralności** (MIC: Low/Medium/High/System – no write up), UAC, GPO (polityki), AppLocker/WDAC (kontrola wykonywania).

## Mechanizmy w systemach rozproszonych
- **usługi katalogowe** jako źródło ról/grup (LDAP, AD) – [[18 LDAP - replikacja i Active Directory]],
- **atrybuty autoryzacji w AAA**: RADIUS zwraca VLAN, ACL, profil (`Filter-Id`), TACACS+ autoryzuje polecenia,
- **kontrola dostępu do sieci**: firewalle (reguły filtrowania = polityka), **802.1X** z dynamicznym VLAN, **NAC**, segmentacja, **Zero Trust** (dostęp oceniany per żądanie na podstawie tożsamości i stanu urządzenia),
- **tokeny z zakresami** – OAuth 2.0 `scope`, JWT z rolami/claims weryfikowane przez bramę API,
- **polityki chmurowe**: AWS IAM (JSON: Effect/Action/Resource/Condition), Kubernetes RBAC + NetworkPolicy + admission controllers (OPA Gatekeeper),
- **kontrola dostępu w bazach**: `GRANT/REVOKE`, row-level security, widoki,
- **Kerberos + autoryzacja** – PAC (_Privilege Attribute Certificate_) w biletach AD zawiera grupy użytkownika.

## Audyt i rozliczalność
Egzekwowanie obejmuje też **rejestrowanie** decyzji (auditd, SELinux AVC, Windows Security Log), okresowe **przeglądy uprawnień** (recertyfikacja kont), wykrywanie nadużyć (SIEM) – [[Zarządzanie Systemami Rozproszonymi/ISO 27001#Zarządanie dostępem (A.9)]].

## Porównanie modeli
| | DAC | MAC | RBAC | ABAC |
|---|---|---|---|---|
| Kto ustala prawa | właściciel | administrator/system | administrator (role) | autor polityk |
| Podstawa decyzji | tożsamość, ACL | etykiety bezpieczeństwa | przynależność do roli | atrybuty + kontekst |
| Elastyczność | wysoka | niska | średnia | bardzo wysoka |
| Ochrona przed trojanami | ✘ | ✔ | częściowa | zależna od polityki |
| Zarządzanie w dużej organizacji | trudne | scentralizowane | łatwe | złożone polityki |
| Przykłady | UNIX, NTFS | SELinux MLS, AppArmor | K8s RBAC, AD grupy | XACML, AWS IAM Conditions, OPA |
