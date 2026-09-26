
# Plan implementacji
1. Uprzątnąć podejście bazowe i ES tak aby moduły z wyjątkiem tych zewnętrznych były takie same i listę endpointów aby były takie same dla wszystkich rozwiązań, skrypty do budowania obrazów dockerowych
2. Na modułach z punktu 1. zrobić podejście ES serverless (narazie wszędzie olewamy CQRS)
3. Napisać wszystkie testy które będą wykorzystywane, dla przewidzianych scenariuszy, przeprowadzić wstępne testy (bez skalowania, nie licząc FaaS bo tam chyba z automatu sobie azure to robi? nie wiem czy lokalnie, a tak na ten moment wystarczy)
4. Odpowiednia implementacja dockerowych rozwiązań tak aby działały na azure containers i się skalowały, upewnić się że FaaS też się skaluje jak powinien, dashboard do wykorzystywanych zasobów, czasów na odpowiedzi itp.
5. Testy finalne,  dla pojedyńczych instancji (bez skalowania) i ze skalowaniem.

# Plan pisemny
todo