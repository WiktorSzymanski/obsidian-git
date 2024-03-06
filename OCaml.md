#SRC #Sem1 #MBP #lab

www.ocaml.org

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
