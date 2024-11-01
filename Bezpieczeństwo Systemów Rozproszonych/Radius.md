---
up:
tags: BezpieczeństwoSystemówRozproszonych
---
# Radius
---
to protokół AAA (Authentication-Authorization-Accounting). Zapewnia scentralizowane uwierzytelnianie, autoryzację oraz rozliczanie użytkowników w środowisku sieciowym.

Jest powszechnie wspierany w urządzeniach sieciowych takich jak:
- routery
- przełączniki
- punkty dostępowe WiFi
- brmy VPN
- itp.

Najpopularniejszą implementacją RADIUS jest [[FreeRADIUS]]. 

### 1. Radius to:
- **4. scentralizowany system implementujący model bezpieczeństwa AAA**
  - **Uzasadnienie**: RADIUS (Remote Authentication Dial-In User Service) to protokół, który implementuje scentralizowany model bezpieczeństwa AAA (Authentication, Authorization, Accounting).

### 2. Radius wykorzystuje komunikację sieciową opartą o protokół:
- **3. UDP**
  - **Uzasadnienie**: RADIUS wykorzystuje protokół UDP (User Datagram Protocol) do komunikacji sieciowej. Domyślnie używa portów 1812 dla uwierzytelniania i autoryzacji oraz 1813 dla księgowania.

### 3. Czy dowolne urządzenie może korzystać z serwera radius w celu realizacji modelu bezpieczeństwa AAA?
- **3. tak, każde z urządzeń korzystających z serwera radius musi znać współdzieloną z serwerem radius tajną informację**
  - **Uzasadnienie**: Urządzenia korzystające z serwera RADIUS muszą znać wspólny tajny klucz (shared secret), który służy do uwierzytelniania i zabezpieczenia komunikacji między klientem a serwerem.

### 4. Hasło użytkownika przesyłane w zapytaniu Access-Request protokołu Radius:
- **4. podlega ochronie z użyciem funkcji skrótu MD5**
  - **Uzasadnienie**: W protokole RADIUS hasło użytkownika w zapytaniu Access-Request jest chronione przy użyciu MD5 do hashowania hasła razem z wartością wspólnego tajnego klucza i wartością nonce.

### 5. Skrót NAS oznacza:
- **2. network access server**
  - **Uzasadnienie**: NAS (Network Access Server) to urządzenie, które funkcjonuje jako punkt dostępowy do sieci, umożliwiający użytkownikom końcowym uzyskanie dostępu do zasobów sieciowych.

### 6. TACACS+ to:
- **4. alternatywne rozwiązanie implementujące model bezpieczeństwa AAA**
  - **Uzasadnienie**: TACACS+ (Terminal Access Controller Access-Control System Plus) to protokół, który również implementuje model bezpieczeństwa AAA. Jest często używany jako alternatywa dla RADIUS, zwłaszcza w środowiskach korzystających z urządzeń firmy Cisco.
  
### 7. Które warunki powinno spełnić urządzenie sieciowe do użycia serwera Radius do AAA:

1. **urządzenie powinno podzielić się “sekretem” z serwerem**
	**Uzasadnienie**: Aby urządzenie sieciowe mogło korzystać z serwera RADIUS do realizacji modelu AAA, musi ono znać i dzielić się wspólnym tajnym kluczem (shared secret) z serwerem RADIUS. Jest to kluczowy element bezpieczeństwa, który zapewnia autentyczność i poufność komunikacji między urządzeniem a serwerem.

**Dlaczego odpowiedź 2 nie jest wystarczająca:**

2. **jest wystarczającym, by połączyć się z serwerem za pomocą jego adresu IP**
	**Wyjaśnienie**: Chociaż połączenie z serwerem za pomocą jego adresu IP jest konieczne, nie jest to wystarczające dla bezpiecznej komunikacji i uwierzytelniania. Bez wspólnego tajnego klucza (shared secret), komunikacja nie będzie odpowiednio zabezpieczona przed podsłuchiwaniem i fałszowaniem.