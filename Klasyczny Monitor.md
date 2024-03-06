#SRC #Sem1 #MBP #wyklad 
#TODO z prezki

- wait na zmiennej warunkowej, wątek wychodzi z monitora do Condition Queue. Jedna kolejka na jedną zmienną warunkową i czeka na spełnienie warunków. Jako że wychodzi z "pudełka", pozwala by inny wątek wszeł. Wtedy inny wątek może wykonać instrukcje która wybudzi wątek uśpiony. Do kolejki Entry Queue przechodzi pierwszy wątek z Condition Queue.
- Kolejność Urgent Queue pierwsze, później dopiero ...