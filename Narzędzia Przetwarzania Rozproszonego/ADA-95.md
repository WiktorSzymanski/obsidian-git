---
up: 
tags:
  - NarzędziaPrzetwarzaniaRozproszonego
---


>"Jest fajny tutorial Simona Johnsona"
>"Język trzyma programiste krótko na smyczy"
>"Wykorzystywany przez NASA"

### ADA Basics
---
- Ada nie zwraca uwagi na wielkość liter
- Apostrof ( ' ) jest używany do wyciągania atrybutów
- Ada jest ściśle typowana
- Bardzo rozwiniętny element pakietowości i ich widoczności
- Nazwa pliku musi być zgodna z nazwą jednostki kompilacji (procedurą, funkcją)

#### Operatory
	:=  Podstawienia
	  =  Porównaina
	/=  Nierówności
	mod Modulo
	rem
	abs
	**
	..  Zakresu

#### Typy
##### Typowanie zmiennej
``` ADA
i : Integer
```

- zdefiniowanie typu INT jako new Integer, sparwia że nie można podstawić zmiennej INT do zmiennej Integer.
- Nie może wystąpić underflow/overflow, w takiej sytacji `wystąpił` by wyjątek

##### String
- String jest indeksowany od jedynki, np. `String(1 .. 20)`. W przypadku tablic ideksowanie jest od dowolnej wartości.

##### Array
``` ADA
arr : array (0 .. 30)
```
`Integer range <>` deklaracja z nie znanym zakresem, trzeba później ten zakres zdefiniować.

##### Record
- odpowiednik `struct` w C.

#### Semantyka
- Osobno część deklaracyjna i przetwarzanie
``` ADA
procedure <NAME> is
	declare
begin
	statement
end <NAME>;
```

##### Konstrukcje warunkowe
	Trzeba jawnie domykać konstrukcje warunkowe
- if
- case
``` ADA
case value is
	when 1..4 => value_ok := 1;
	when 5 | 6 | 7 => null;
end case;
```
- pętle
	- `exit when expresion` odpowiednik `break`
	- brak odpowiednika `do while`, `exit when` rekompensuje ten brak
``` ADA
loop
	statement
end loop;

while expresion loop
	statement
end loop;

for ident in <(reverse) range> loop
	statement
end loop;
```
- procedury - funkcje nie zwracające wartości
##### Przesyłanie argumentów
- in - nie wolno pod niego nic podstawić, tylko z niego czytamy
- out - trzeba pod niego coś podstawić, nie czytamy z niego
- Parametry domyślne
- Dopasowanie pozycyjne i po nazwach
``` ADA
Create(File_Handle, Inout_File, "text.file");
Create(File => File_Handle, Name => "text.file");
```

### Współbieżność
---
