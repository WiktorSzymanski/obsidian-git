---
up: "[[DDD]]"
---
# Domain Model *(ang. Model Domeny)*
---
> Termin ten odnosi się do [[Model|Modelu]] w danej [[Domain|Domenie]]. Tworząc oprogramowanie mające rozwiązać problemy w pewnej [[Domain|domenie]], rozwiązanie musi operować w tej [[Domain|domenie]] i być z nią zgodne. Dlatego aby to osiągnąć trzeba zadecydować które aspekty [[Domain|domeny]] są **istotne**. Dlatego utworzenie **modelu domeny**, który reprezentuje te aspekty, ma kluczowe znaczenie dla osób zaangażowanych w poznanie, zrozumienie i symulację rzeczywistej domeny.

### Część rzeczywistej domeny należy uznać za istotną gdy:
---
- Bezpośrednio lub pośrednio pomaga w rozwiązaniu danego problemu
- Musi być brana pod uwagę, nawet jeśli nie pomaga w rozwiązaniu problemu (przykładem mogą być przepisy i regulacje których trzeba przestrzegać)

Każdy inny aspekt rzeczywistej domeny powinien być pominięty w **modelu domeny**.

> Celem jest posiadanie użytecznego modelu, a nie w pełni poprawnego.

#### Przykład                       
![[globe-showing-europe-africa.svg|right|150]] Przykładem domeny może być nasza planeta. Posiada ona wiele różnych modelów, jednym z nich może być mapa. Jest wiele różnych map (**modeli domeny** jaką jest ziemia) i każda może być przystosowana do czego innego. 
![[world-map.svg|left|150]] Ta na przykład mogła by służyć do wyznaczania kierunków w jakich należy wyruszyć by wypłynąć z europy czy afryki w danym kierunku.
