#SRC #Sem1 #MBP #lab

www.ocaml.org

Projekt zaliczeniowy idea:
- Jakaś gra
- Funkcje robione z funkcji jako ciekawa zaleśność języka (różne poziomy umiejętności)
- Referencja funkcji jako atak który można zmieniać
- 

OCaml to [[Język Funkcyjny]].

##### Interpretuje funkcję jak zmienne. Przykład: uznaje int_of_float jako pierwszy argument funkcje. 
``` ocaml
fun1 int_of_float x 2;;
```

##### Funkcje rekurencyjne muszą mieć słowo kluczowe _rec_.
``` ocaml
let f m n = m n;;
let g = f (fun x y -> x + y) 10;; // g staje się funkcją jednoargumentową 
```

##### Można definiować funkcje wewnętrze i zmienne lokalne, które zamykamy w _scope_ funkcji.
``` ocaml
let average a b =
	let sum x y = x + y in
	let d = 2 in
	(sum a b) / d;;
```

##### Referencje 
``` ocaml
let y = ref 1;;
```
Do wyciągnięcia wartości wstaźnika używamy `!`.

``` ocaml
!y + 10
```
Typ wyrażenia bez `!` to `unit = ()`. Nie nadużywać referencji przy pisania programów funkcyjnych.

``` ocaml
y := 10
```
Przypisanie do referencji wartosci.

Referencja może wskazywać na funkcję, a podmiana tej funkcji może mieć miejsce jeśli funkcje mają te same typy.

W języku funkcyjnym można podać do funkcji mniej argumentów niż wymaga i otrzymamy nową funkcję. Można też napisać funkcję która przyjmuje więcej argumentów niż oczekuje.
``` ocaml
let f x =
	let g = fun a -> a - z in
	g;;
```

Operatory to tez funkcje więc moża je przekazywać jako argumenty.

`in` oznacza zagnieżdzanie funkcji we wunkcji.

`1::2::3::[];;`  tworzy liste `[1; 2; 3;]`
`@` oznacza konkatenacje list

#### Dopasowanie do wzoraca
``` ocaml
match l with
  [] -> 0
| [a] -> a
| [a;b] -> a+b;;
```
Przekazanie do `match` większej ilości elementów jak `match` pozwala, generuje wyjątek (wyjątkowa sytacja dla tego języka). Aby uniknąć tych wyjątków/warningów warto na koniec `match` dodać 
``` ocaml
| _ -> -1;;
```
aby obsłóżyć jakikolwiek inny przypadek.

##### Zad 1
``` ocaml
let maczek l =
  let timesListIf = fun a ->
    let times3 = fun x -> x * 3 in
    List.map times3 a in
  match l with
    [] -> []
  | [a] -> []
  | [a;b] -> []
  | _ -> times3 l;;
```

##### Zad 2 - Napisać funkcję działającą jak `@`.
``` ocaml
let rec append = fun l1 l2 ->
  match l1 with
  | [] -> l2
  | h :: t -> h :: append t l2;;
```

#### Printf
``` oCaml
Printf.printf "%i" 10;; (*int*)
Printf.printf "%s" "10";; (*string*)
Printf.printf "%c" '10';; (*char*)
```
Jeśli nie chcemy używać "kropkowego" _syntax_-u możemy użyć `open Printf`.
``` ocaml
open Printf;;
printf "%i" 10;;
```

#### Typy
``` ocaml
type foo =
   Liczba of int
 | Tekst of string
 | Rz of float
 | Nic;;

let x : foo = Nic;;
let y = Liczba(100);;
let z = Tekst("ala");;
```

``` ocaml
type 'a btree = 
   Leaf of 'a (*zmienn typu aby drzew było uniwersalne*)
 | Empty
 | Node of 'a btree * 'a btree;;

let sosna = Node (Leaf 10, Node (Leaf 20, Empty));;
```
`Empty` aby można rozróżnić prawy od lewego (node/liścia).  Można typowi nadać zmienną typu `'a` dzięki czemu typ nie ogranicza się do jednego typu danych.