---
up: 
tags: BezpieczeństwoSystemówRozproszonych
---
# Pluggable Authentication Modules
---
>jest narzędziem będącym systemem modularnego uwierzytelniania szeroko stosowanym w środowisku Linux, posiadającym bardzo dużą liczbę dostępnych modułów rozszerzających. Rozdziela logikę aplikacji od polityki i mechanizmów kontroli dostępu. Koncepcja i pierwsza implementacja PAM została opracowana dla systemu Solaris w 1995 roku, a po kilku latach przeniesiona na Linux-a.

System PAM to zestaw bibliotek, modułów, wtyczek, które są wykorzystywane do uwierzytelniania użytkowników w systemie. Odpowiednia biblioteka jest stosowana gdy polityka uwierzytelniania dla danej aplikacji wywołuję procedurę uwierzytelniającą. Moduły też określają dodatkowe wymagania i zadania systemu uwierzytelniającego, np. ograniczenia czasowe do pewnych godzin czy określenie innego źródła danych (baza [[LDAP]], SQL czy inne).

## Komponenty PAM
Moduły systemu PAM mogą realizować funkcje w czterech obszarach polityki (_management groups_):
- **_auth_** - Moduł **zarządzania uwierzytelnianiem** realizuje dwa zadania. Ważniejsze, uwierzytelnianie użytkowników, i drugorzędne, ustalanie uprawnień użytkownika, np. na podstawie przynależności do grup. Na polecenie modułu _auth_ aplikacja pyta użytkownika o dane uwierzytelniające. 
- **_account_** - Moduł **zarządzania kontami** odpowiada za weryfikację, czy można podjąć próbę dostępu do konta użytkownika w danych okolicznościach. Mogą weryfikować porę dnia, sposób dostępu (np. zdalny lub lokalny), obciążenie systemu, a nawet ważność konta i reguły dotyczące wygaśnięcia hasła. Komponenty typu _account_ nie mają związku z samym procesem uwierzytelniania. 
- **_password_** - Moduł **zarządzania hasłami** umożliwia użytkownikowi zmiany hasła lub innej metody uwierzytelniania. Zajmują się też weryfikacją poprawności i złożoności zmienionych poświadczeń. 
- **_session_** - Moduły **zarządzania sesjami** mogą zarządzać sesjami użytkowników, odpowiadają za ustalenie sesji oraz "porządki" podczas jej zakończenia. Do elementów tworzenia sesji mogą należeć takie zadania jak ustalanie zmiennych środowiskowych, uruchomienie `chroot` czy ograniczanie dostępnych zasobów. Dodatkowo odpowiadają za wyświetlanie powiadomień (jak np. `motd`).

Każdy z powyższych obszarów może być definiowany w indywidualny sposób dla każdej aplikacji, a możliwości polityki są praktycznie nieograniczone. Zależą wyłącznie od zastosowanych modułów PAM.

#### Przykładowe moduły PAM
- **access** – moduł określający kto ma mieć dostęp do systemu oraz w jaki sposób może uzyskać dostęp
- **cracklib** oraz **pwquality** – moduły sprawdzające jakość hasła użytkownika,
- **pwhistory** – moduł sprawdzający, jak bardzo hasło różni się od poprzednich,
- **anonymous access** – moduł umożliwiający stworzenie anonimowego dostępu do serwera FTP,
- **resource limits** – określa limity wykorzystania systemu przez użytkownika; limitami takimi można objąć wykorzystanie pamięci operacyjnej dla pojedynczego procesu i dla wszystkich procesów użytkownika, maksymalny czas procesora, maksymalną liczbę logowań użytkownika i wiele innych,
- **radius session** – uwierzytelnianie użytkownika przy wykorzystaniu zdalnego serwera RADIUS,
- **time control** – określa dostęp użytkownika do systemu w zależności od czasu, w którym próbuje on uzyskać dostęp,
- **kerberos** – uwierzytelnianie użytkownika przy wykorzystaniu systemu [[Kerberos]],
- **smart card** – uwierzytelnianie użytkownika na podstawie karty chipowej,
- **one-time password** – uwierzytelnianie użytkownika na podstawie jednorazowych haseł,
- **SQL database** – uwierzytelnianie użytkownika na podstawie wpisów w bazie danych; istnieją wtyczki do większości baz danych,
- **SecurID** – uwierzytelnianie użytkownika poprzez tokeny (potrzebny jest do tego serwer, w którym tokeny będą zarejestrowane),
- **voiceauth** – uwierzytelnianie użytkownika na podstawie jego głosu,
- **LDAP** – uwierzytelnianie użytkownika na podstawie wpisów w bazie [[LDAP]].