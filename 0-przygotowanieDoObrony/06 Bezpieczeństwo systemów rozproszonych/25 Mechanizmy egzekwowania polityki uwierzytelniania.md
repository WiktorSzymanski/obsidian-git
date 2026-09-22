---
tags:
  - obrona
up: "[[Mapa zagadnień]]"
zagadnienie: 25
---
# 25. Mechanizmy egzekwowania polityki uwierzytelniania
---
> **Uwierzytelnianie** (_authentication_) to weryfikacja **deklarowanej tożsamości** podmiotu (użytkownika, usługi, urządzenia). **Polityka uwierzytelniania** określa, **kto**, **czym** (jakie poświadczenia i czynniki), **kiedy** i **skąd** może się uwierzytelnić. Mechanizmy egzekwowania to komponenty systemu, które te reguły **wymuszają**.

Uwaga na pojęcia (**AAA**): **identyfikacja** (kim twierdzę, że jestem) → **uwierzytelnianie** (dowód) → **autoryzacja** (co mogę – [[27 Mechanizmy egzekwowania polityki kontroli dostępu]]) → **rozliczanie** (_accounting_, co zrobiłem).

## Czynniki uwierzytelniania
| Czynnik | Przykłady | Zagrożenia |
|---|---|---|
| **coś, co wiesz** | hasło, PIN, odpowiedź na pytanie | zgadywanie, phishing, wycieki, keyloggery |
| **coś, co masz** | token OTP, karta inteligentna, klucz FIDO2, telefon (SMS/aplikacja), certyfikat | kradzież, SIM swap, MitM w czasie rzeczywistym |
| **coś, czym jesteś** | odcisk palca, twarz, tęczówka, głos | fałszywe akceptacje, brak możliwości zmiany |
| (dodatkowo) **gdzie jesteś / jak się zachowujesz** | adres IP, geolokalizacja, dynamika pisania | – |

- **MFA / 2FA** – co najmniej dwa czynniki **różnych** kategorii (hasło + hasło to nie 2FA).
- **Uwierzytelnianie adaptacyjne / ryzyka** – dodatkowy czynnik wymagany tylko przy nietypowej sytuacji (nowe urządzenie, kraj).

## Mechanizmy – przegląd
### 1. Hasła i ich polityka
- Przechowywanie **wyłącznie jako skróty z solą** funkcjami celowo wolnymi: **bcrypt**, **scrypt**, **Argon2**, PBKDF2 (sól chroni przed tablicami tęczowymi, koszt – przed atakiem słownikowym offline).
- Polityka: minimalna długość (NIST SP 800-63B: ≥ 8, zalecane dłuższe frazy), **sprawdzanie haseł na listach wycieków** zamiast wymuszania złożoności i okresowej zmiany, blokada/opóźnianie po nieudanych próbach (ochrona przed brute force), historia haseł.
- Egzekwowanie w Linuksie: **PAM** – `pam_pwquality` (jakość), `pam_pwhistory` (historia), `pam_faillock` (blokada konta), `pam_unix` (hasła z `/etc/shadow`) – [[Bezpieczeństwo Systemów Rozproszonych/PAM]].

### 2. Hasła jednorazowe (OTP)
- **HOTP** (RFC 4226) – oparte na liczniku: $HOTP(K, C) = Truncate(HMAC\text{-}SHA1(K, C)) \bmod 10^d$; licznik $C$ zwiększany przy każdym użyciu, serwer akceptuje okno kilku kolejnych wartości.
- **TOTP** (RFC 6238) – oparte na czasie: $C = \lfloor (T - T_0) / X \rfloor$, zwykle $X = 30$ s (Google Authenticator); wymaga synchronizacji zegarów.
- Listy haseł jednorazowych (S/KEY – łańcuch skrótów Lamporta), SMS/e-mail (słabe – przechwycenie).
- Egzekwowanie: `pam_google_authenticator`, `pam_oath`, serwer RADIUS z modułem OTP.

### 3. Wyzwanie-odpowiedź (_challenge-response_)
Serwer wysyła losowe **wyzwanie** (nonce), klient odpowiada funkcją sekretu i wyzwania (HMAC, podpis) → sekret nie jest przesyłany, a odpowiedzi nie da się powtórzyć (**ochrona przed replay**). Przykłady: HTTP Digest, CHAP, NTLM, SCRAM (SASL), FIDO2.

### 4. Uwierzytelnianie HTTP
Basic (hasło w Base64 – tylko przez TLS), Digest (skróty MD5 z nonce) – [[Bezpieczeństwo Systemów Rozproszonych/HTTP Authentication]].
**Podatności HTTP Digest** (uzupełnienie `#TODO`):
- **atak MitM / downgrade** – pośrednik podszywa się pod serwer i żąda uwierzytelnienia **Basic** → hasło jawnym tekstem (brak uwierzytelnienia serwera),
- **atak słownikowy offline** – przechwycona odpowiedź (znane nonce, realm, URI) pozwala sprawdzać hasła lokalnie (MD5 jest szybkie),
- serwer przechowuje **HA1 = MD5(user:realm:password)** – odpowiednik hasła (kradzież bazy = możliwość logowania bez znajomości hasła),
- ochrona tylko uwierzytelnienia – brak integralności/poufności treści (opcjonalne `qop=auth-int` rzadko wspierane), MD5 przestarzałe,
- w praktyce: **TLS + Basic/formularz** lub mechanizmy tokenowe.

### 5. Certyfikaty X.509 i PKI
- Para kluczy asymetrycznych; **certyfikat** wiąże klucz publiczny z tożsamością, podpisany przez **urząd certyfikacji (CA)** → **łańcuch zaufania** do zaufanego CA głównego.
- Uwierzytelnienie = dowód posiadania klucza prywatnego (podpis wyzwania).
- Weryfikacja: podpis CA, okres ważności, nazwa (CN/SAN), rozszerzenia (Key Usage, EKU), **unieważnienie** (CRL, OCSP, OCSP stapling).
- **TLS**: domyślnie uwierzytelniany tylko **serwer**; **mTLS** (wzajemny TLS) – serwer żąda certyfikatu klienta (`CertificateRequest`), stosowany między usługami (service mesh), w VPN, EAP-TLS.
- **Karty inteligentne / PIV** – klucz prywatny w chronionym układzie (nieeksportowalny).

### 6. Kerberos (MIT, v5 – RFC 4120)
> Protokół uwierzytelniania w sieci niezaufanej, oparty na **zaufanej trzeciej stronie** (KDC) i **kryptografii symetrycznej**. Daje **SSO** w obrębie domeny (realm) oraz **wzajemne uwierzytelnienie** klienta i usługi.

**Pojęcia**: **principal** (tożsamość, np. `jan@FIRMA.PL`, `HTTP/www.firma.pl@FIRMA.PL`), **realm** (domena), **KDC** = **AS** (_Authentication Server_) + **TGS** (_Ticket Granting Server_), **bilet** (_ticket_), **TGT** (_Ticket Granting Ticket_), **klucz sesyjny**, **authenticator**, **keytab** (plik z kluczami usługi).

Oznaczenia: $K_c$ – klucz klienta (z hasła), $K_{tgs}$ – klucz TGS, $K_s$ – klucz usługi, $K_{c,tgs}$ i $K_{c,s}$ – klucze sesyjne.
1. **AS-REQ**: klient → AS: $c$, $tgs$, znacznik czasu (pre-authentication: $\{ts\}K_c$).
2. **AS-REP**: AS → klient: $\{K_{c,tgs}, tgs, czas\}K_c$ oraz **TGT** $= \{c, adres, K_{c,tgs}, ważność\}K_{tgs}$.
   Klient odszyfrowuje część $K_c$ kluczem z hasła → zna $K_{c,tgs}$; **hasło nie jest dalej potrzebne** (SSO).
3. **TGS-REQ**: klient → TGS: $s$, **TGT**, **authenticator** $\{c, ts\}K_{c,tgs}$.
4. **TGS-REP**: TGS → klient: $\{K_{c,s}, s, czas\}K_{c,tgs}$ oraz **bilet usługi** $= \{c, adres, K_{c,s}, ważność\}K_s$.
5. **AP-REQ**: klient → usługa: bilet usługi, authenticator $\{c, ts\}K_{c,s}$.
6. **AP-REP** (opcjonalnie, uwierzytelnienie wzajemne): usługa → klient: $\{ts\}K_{c,s}$ (lub $ts+1$).

**Własności**:
- hasło nigdy nie jest przesyłane; usługa nie kontaktuje się z KDC (weryfikuje bilet swoim kluczem),
- **ochrona przed powtórzeniem**: znaczniki czasowe w authenticatorach (tolerancja **±5 min** – wymagana **synchronizacja zegarów**, NTP) + pamięć podręczna authenticatorów,
- bilety mają ograniczoną ważność (TGT zwykle 10 h), możliwe odnawianie i delegacja (_forwardable_),
- **wady**: KDC to SPoF i cel ataku (replikacja KDC), ataki na słabe hasła (AS-REP roasting, Kerberoasting), Pass-the-Ticket, _Golden Ticket_ (kradzież klucza `krbtgt`),
- zaufanie między realmami (_cross-realm_),
- **integracja**: GSS-API/SASL GSSAPI (LDAP, SSH, NFSv4, HTTP SPNEGO/Negotiate), `pam_krb5`/SSSD, **Active Directory** ([[18 LDAP - replikacja i Active Directory]]), FreeIPA.

### 7. RADIUS, TACACS+ i scentralizowane AAA
- **RADIUS** (RFC 2865/2866): klient **NAS** (router, AP, VPN) przekazuje poświadczenia do serwera RADIUS; **UDP 1812/1813**, współdzielony sekret, hasło ukryte przez MD5 z sekretem; atrybuty (VLAN, ACL) zwracane jako autoryzacja – [[Bezpieczeństwo Systemów Rozproszonych/Radius]], [[Bezpieczeństwo Systemów Rozproszonych/FreeRADIUS]].
- **TACACS+** (Cisco): TCP 49, szyfrowanie całego pakietu, rozdzielenie A/A/A, autoryzacja poleceń na urządzeniach.
- **Diameter** – następca RADIUS (sieci komórkowe).

### 8. 802.1X / EAP – kontrola dostępu do portu sieci
- **Suplikant** (klient) – **Uwierzytelniacz** (przełącznik/AP) – **Serwer uwierzytelniania** (RADIUS).
- Port **zablokowany** (przepuszcza tylko EAPOL) do czasu udanego uwierzytelnienia; uwierzytelniacz przekazuje EAP w RADIUS.
- Metody **EAP**: **EAP-TLS** (certyfikaty obu stron – najsilniejsza), **PEAP** i **EAP-TTLS** (tunel TLS z certyfikatem serwera, wewnątrz hasło, np. MSCHAPv2), EAP-FAST; słabe: EAP-MD5.
- Wynik: dynamiczny VLAN, klucze szyfrowania WPA2/3-Enterprise.

### 9. Uwierzytelnianie w aplikacjach webowych i API
- **sesje** z ciasteczkiem (`HttpOnly`, `Secure`, `SameSite` – [[28 Polityki Same Origin i Same Site]]), regeneracja ID sesji po logowaniu (ochrona przed fiksacją),
- **tokeny** – JWT (podpisane: nagłówek.ładunek.podpis; weryfikacja podpisu, `exp`, `aud`, `iss`), tokeny nieprzezroczyste,
- **OAuth 2.0** (RFC 6749) – framework **autoryzacji delegowanej**:
  - role: właściciel zasobu, klient, serwer autoryzacji, serwer zasobów,
  - przepływy: **authorization code + PKCE** (aplikacje web/mobilne), **client credentials** (usługa-usługa), device code; (implicit i password – przestarzałe),
  - **access token** (krótki), **refresh token**,
- **OpenID Connect** – warstwa **uwierzytelniania** na OAuth 2.0: **ID Token** (JWT z `sub`, `iss`, `aud`, `nonce`), `/userinfo`, discovery – [[26 Jednokrotne uwierzytelnianie - SSO]],
- **FIDO2/WebAuthn** (passkeys) – kryptografia asymetryczna, klucz powiązany z domeną → **odporność na phishing**, brak sekretu po stronie serwera,
- **klucze API**, podpisy żądań (AWS SigV4, HMAC).

### 10. Uwierzytelnianie w systemach i usługach
- **PAM** – modularny stos z flagami sterującymi (`required`, `requisite`, `sufficient`, `optional`) – kolejność modułów wyraża politykę,
- **NSS / SSSD** – źródła tożsamości (LDAP, AD) z buforowaniem poświadczeń offline,
- **SSH** – klucze publiczne (`authorized_keys`, certyfikaty SSH podpisane przez CA), `AuthenticationMethods publickey,keyboard-interactive` (wymuszenie 2FA), Kerberos (GSSAPI),
- **LDAP bind** – simple (tylko z TLS) lub SASL (GSSAPI, EXTERNAL z certyfikatem),
- **usługi między sobą** – mTLS, tokeny SPIFFE/SPIRE, Kerberos keytab.

## Elementy egzekwowania polityki
- **punkt egzekwowania** (PAM, NAS/przełącznik, brama API, reverse proxy z OIDC) i **punkt decyzji** (KDC, RADIUS, IdP),
- **centralizacja** tożsamości (LDAP/AD/IdP) – spójna polityka, łatwe blokowanie kont,
- **ograniczenia kontekstowe**: pora dnia (`pam_time`), źródło (`pam_access`), urządzenie (Conditional Access),
- **blokady i limitowanie** prób (lockout, CAPTCHA, rate limiting),
- **ochrona kanału** – TLS (poświadczenia nie mogą iść jawnym tekstem),
- **zarządzanie cyklem życia**: nadawanie, wygaszanie, odbieranie (offboarding), rotacja sekretów i kluczy,
- **rejestrowanie i monitorowanie** logowań (SIEM), alarmy o anomaliach,
- **zasada**: silniejsze uwierzytelnianie dla kont uprzywilejowanych (PAM – _privileged access management_, MFA dla administratorów).

## Zobacz też
- [[26 Jednokrotne uwierzytelnianie - SSO]]
- [[Technologie internetowe w przetwarzaniu rozproszonym/WS-Security]], [[Bezpieczeństwo Systemów Rozproszonych/SSL & TLS]]
