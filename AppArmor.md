---
up: 
class: BSR
---
# AppArmor
---
### 1. Polityka AppArmora pozwala dla konkretnej aplikacji zdefiniować:

3. prawa dostępu aplikacji do poszczególnych zasobów
	**Uzasadnienie**: AppArmor pozwala na określenie, do których zasobów systemowych aplikacja ma dostęp (np. plików, sieci).

4. wybrane systemowe uprawnienia administracyjne (capabilities) przydzielane procesowi aplikacyjnemu
	**Uzasadnienie**: AppArmor pozwala na definiowanie specjalnych uprawnień administracyjnych (capabilities) dla aplikacji, co daje kontrolę nad tym, jakie operacje może wykonywać aplikacja.
### 2. W przypadku środowiska AppArmor:

1. każdą politykę można definiować "od zera" zwykłym edytorem tekstowym, bez potrzeby wykorzystywania dedykowanych narzędzi AppArmora
	**Uzasadnienie**: Polityki AppArmora są plikami tekstowymi, które można edytować ręcznie za pomocą dowolnego edytora tekstowego.
1. możliwe jest uruchomienie dowolnego istniejącego profilu w trybie uczenia się, w którym restrykcje zdefiniowane w tym profilu nie są egzekwowane
	**Uzasadnienie**: AppArmor posiada tryb "complain", który umożliwia uruchomienie profilu w trybie uczenia się, gdzie restrykcje nie są wymuszane, a zamiast tego generowane są raporty.
### 3. Rozwiązanie AppArmor to mechanizm dedykowany do zabezpieczania:

- **4. tylko wybranych aplikacji**
    - **Uzasadnienie**: AppArmor zabezpiecza wybrane aplikacje poprzez tworzenie dla nich profili, które definiują, co aplikacje mogą robić.
### 4. Rozwiązanie AppArmor nie jest koncepcyjnie modelem zabezpieczeń:

- **2. IBAC (Identification-Based Access Control)**
    
    - **Uzasadnienie**: AppArmor działa na zasadzie MAC (Mandatory Access Control), a nie IBAC, który opiera się na tożsamości użytkowników.
- **4. DAC (Discretionary Access Control)**
    
    - **Uzasadnienie**: AppArmor stosuje MAC, w którym dostęp jest kontrolowany na podstawie polityk bezpieczeństwa, a nie decyzji użytkowników, jak w przypadku DAC.
- RSBAC (Rule Set Based Access Control) jest modelem zabezpieczeń, który koncepcyjnie przypomina MAC (Mandatory Access Control), podobnie jak AppArmor. Oba systemy działają na zasadzie wymuszania polityk bezpieczeństwa, które są niezależne od użytkownika i narzucane przez administratora systemu. RSBAC, podobnie jak MAC, jest systemem, w którym administratorzy definiują zasady, a system wymusza te zasady niezależnie od preferencji użytkowników. AppArmor działa w podobny sposób, definiując polityki bezpieczeństwa (profile) dla aplikacji, które są następnie wymuszane przez system.
### 5. Profil AppArmor to:

- **2. plik tekstowy zawierający uprawnienia dla zabezpieczanej aplikacji**
    - **Uzasadnienie**: Profil AppArmor to plik tekstowy, który zawiera definicje uprawnień i restrykcji dla danej aplikacji.
### 6. Profil AppArmor

- **3. część polityk AppArmor związana z konkretną aplikacją**
    
    - **Uzasadnienie**: Profil to konkretna część polityki, która jest przypisana do danej aplikacji, definiując jej uprawnienia.
- **4. plik tekstowy definiujący uprawnienia wybranej aplikacji**
    
    - **Uzasadnienie**: Profil jest zapisany w pliku tekstowym i określa, jakie operacje może wykonywać dana aplikacja.    
### 6. Profil AppArmor

- **3. część polityk AppArmor związana z konkretną aplikacją**
    
    - **Uzasadnienie**: Profil to konkretna część polityki, która jest przypisana do danej aplikacji, definiując jej uprawnienia.
- **4. plik tekstowy definiujący uprawnienia wybranej aplikacji**
    
    - **Uzasadnienie**: Profil jest zapisany w pliku tekstowym i określa, jakie operacje może wykonywać dana aplikacja.
    
### 8. Tryb działania AppArmor o nazwie "enforce" powoduje:

- **2. wymuszanie profilu uprawnień dla wybranych aplikacji**
    - **Uzasadnienie**: W trybie "enforce" AppArmor wymusza wszystkie restrykcje zapisane w profilu aplikacji, uniemożliwiając wykonywanie niedozwolonych operacji.

### 9. Czy należy zawsze zmodyfikować aplikacje, które mają podlegać ochronie rozwiązaniem AppArmor:

- **2. nie, nie zawsze trzeba przystosowywać aplikację do współpracy z AppArmor**
    - **Uzasadnienie**: Zazwyczaj nie jest konieczne modyfikowanie samej aplikacji, ponieważ AppArmor działa na poziomie systemu operacyjnego i kontroluje dostęp do zasobów na podstawie profili.

### 10. Change Hat:

**2. pozwala na dynamiczną zmianę uprawnień w trakcie działania zabezpieczonego programu**
    - **Uzasadnienie**: "Change Hat" jest funkcją w AppArmor, która umożliwia aplikacji dynamiczne przełączanie się między różnymi profilami (kapeluszami) w trakcie działania.
**4. to nazwa funkcji API rozwiązania AppArmor**
	- **Uzasadnienie**: AppArmor posiada funkcje API `aa_change_hat`, `aa_change_hatv` i `aa_change_hat_vargs`, które implementują tę funkcjonalność.
### 11. Implementacją sandboxa jest

- **1. Polityka AppArmor**
    - **Uzasadnienie**: Polityki AppArmor mogą być używane do tworzenia piaskownic (sandboxów) dla aplikacji, ograniczając ich zdolność do wykonywania operacji poza zdefiniowanym zestawem uprawnień.