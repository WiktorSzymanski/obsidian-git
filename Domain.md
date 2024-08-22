---
up: "[[DDD]]"
---
# Domain *(ang. Domena)*
---
> to pewna określona sfera wiedzy, wpływów lub działalności. Jest to **obszar danego tematu lub środowisko** (czasami oba), do którego jest przygotowywane oprogramowanie.

### Domena jest częścią świata naturalnego i zwykle składa się z:
- *Terms* - terminy, które mają określoną definicję w sferze wiedzy, wpływów lub działalności
- *Objects* - obiekty które mają określone stany w określonych punktach w czasie
- *Operations* - operacje tworzące obiekty, zmieniające ich stany lub kończące je
- *Invariants*, niezmienne które muszą być egzekwowane przez zmiany stanów

#### Przykład:
---
W **domenie** "bankowości" istnieją:
- Terminy takie jak `saldo`, `transakcja`, `aktywa`, które mają precyzyjne definicje
- Obiekty takie jak `klient`, `konto`, `wydatki`, mogące mieć różne stany w różnych punktach w czasie
- Operacje jak `utworzenie konta`, `przelew`, `wpłata`, powodujące zmiany stanu
- Niezmienne, takie jak `użytkownicy nie mogą przekroczyć limitów kredytowych`, co musi być egzekwowane bez względu na operację

Podczas tworzenia domeny należy unikać ponownego wymyślania **domeny**. Jeśli uważasz, że program potrzebuje dodatkowych, nowych **terminów** lub **operacji**, warto skonsultować się z ekspertem w danej domenie. Istnieje szansa, że to co chcesz dodać istnieje w domenie lecz jeszcze tego nie odkryłeś.

Gdy **domena** jest bardzo obszerna warto podzielić ją na **pod-domeny**. Tutaj również warto skonsultować się z ekspertem w danej domenie czy takowe już nie są dobrze zdefiniowane.