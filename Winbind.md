---
up: "[[Samba]]"
class: ZSK
---
# Winbind
---

>to komponent [[Samba|samby]] umożliwiający integrację systemów uniksowych z usługą katalogową firmy Microsoft, [[Active Directory]]. Jego główną rolą jest zarządzanie kontami użytkowników i grupami z [[Active Directory|AD]], co pozwala na jednolite logowanie ([[SSO]]) w zróżnicowanych 
>(**heterogenicznych**)  środowiskach.

Ciekawym rozwiązaniem Winbind jest _caching_ poświadczeń. Dzięki przechowywaniu w pamięci podręcznej informacji o użytkownikach i grupach, pozwala zwiększyć wydajność podczas uwierzytelniania, a nawet pozwolić na nie w przypadku chwilowej niedostępności do kontrolera domeny [[Active Directory|AD]].