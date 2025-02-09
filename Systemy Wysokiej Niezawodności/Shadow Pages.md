---
up: "[[Przywracanie stanu]]"
tags:
  - SystemyWysokiejNiezawodności
---
# Shadow Pages
---
>Technika często stosowana na poziomie sprzętowym przy dostępie do pamięci. Niektóre serwery mają wbudowany taki mechanizm działania, gdzie przy modyfikacji strony pamięci tworzona jest **kopia robocza**, a oryginał jest zapamiętywany jako tzw. **Shadow Page**. W momencie gdy **kopia robocza** zostanie pomyślnie zapisana, **shadow page** zostaje usuwany, a w przypadku gdy zapis się nie powiedzie przywracany jest stan za pomocą **shadow page-a**.

![[Pasted image 20250203180909.png]]

**Wadą** tego rozwiązania rozwiązania jest zajmowanie dodatkowego miejsca podczas modyfikacji.
## Twin-Page
---
Szczególny przypadkiem **shadow pages** gdzie cały czas utrzymywane są dwie kopie danych, a operacje na nich wykonywane są **naprzemiennie**. W przypadku konieczności wycofania się z jakiegoś zapisu, możemy tak zrobić, ponieważ **bliźniacza kopia przechowuje poprzedni stan**.

![[Pasted image 20250203181156.png]]