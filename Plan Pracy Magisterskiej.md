1. Skupić się na tym jakie miary, testy i elementy aplikacji. Co chce zbadać jak to oceniać.
	- Co mierzyć?
		1. Opóźnienie
		2. Przepustowość
		3. Wykorzystywanie zasobów
		4. Skalowalność
		5. Odtwarzanie (_recovery_)
		6. Odporność na awarie (_fault tolerance_)
		7. Spójność danych (???)
	- Jak?
		- Różnego natężenia stres testy na jednej instancji (_1. 2. 3._)
		- Różnego natężenia stres testy biorące pod uwagę skalowanie (_1. 2. 3. 4._)
		- _Chaos Tests_ i _Fault Injection Tests_ (_6._)
		- _Recovery Tests_ i _Disaster Recovery Tests_ (_5._)  
	
2. Zastanowić się jakie platformy.
	- dostawcy Amazon, Google i Microsoft pozwalają na wykorzystanie Kotlina stąd jeśli istnieje możliwość można napisać to w ten sposób by nie kod nie był _provider specific_ i pokusić się o przetestowanie wszystkich, jednak nie wydaje mi się że powinno to mieć wpływ na wyniki (przy założeniu, że przetwarzanie dysponuje takimi samymi zasobami). Stąd jeśli miało by to wprowadzić jakiś większy narzut to wydaje mi się, że nie warto.  


3. Co zyskujemy przez różne wyróżnione elementy
	- Jak wpływa na wydajność CQRS (konieczny branch, który działa bez niego)
	- Jak wpływa Snapshot (i to jak często się go wykonuje) na Event Sourcing.
	- Osobne _Query_ i _Command_ mogą pozytywnie wpłynąć na skalowanie  
		
4. Interesujące punkty gdzie podejście się wyróżnia
	- Nie pamiętam do końca o co chodziło, jeśli chodzi o wyróżnianie się na tle innych artykułów naukowych wydaje mi się, że nie ma badania _event sourcing_-u jako _FaaS_

5.  Środowisko
	- Wybrane środowisko/ska chmurowe z aplikacją opartą o _FaaS_ i te z "klasycznym podejściem" postawione w ramach _IaaS_