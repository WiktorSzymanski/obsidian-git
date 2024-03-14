#SRC #Sem1 #MBP #wyklad 
#TODO z prezki https://www.cs.put.poznan.pl/pawelw/teaching/mbp/index.html

- wait na zmiennej warunkowej, wątek wychodzi z monitora do Condition Queue. Jedna kolejka na jedną zmienną warunkową i czeka na spełnienie warunków. Jako że wychodzi z "pudełka", pozwala by inny wątek wszeł. Wtedy inny wątek może wykonać instrukcje która wybudzi wątek uśpiony. Do kolejki Entry Queue przechodzi pierwszy wątek z Condition Queue.
- Kolejność Urgent Queue pierwsze, później dopiero ...

#TODO zaimplementować sobie monitory w C# lub Java