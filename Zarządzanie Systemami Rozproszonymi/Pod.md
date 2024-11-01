---
tags:
  - ZarządzanieSystemamiRozproszonymi
up: "[[Kubernetes]]"
---
# Pod
---
>to najmniejsza jednostka [[Kubernetes]], która uruchamia kontenery. Każdy **Pod** może zawierać jeden lub więcej kontenerów, które współdzielą sieć i woluminy. **Pod**-y są **efemeryczne** - mogą być usuwane i ponownie uruchamiane w zależności od stanu aplikacji i potrzeb klastra. W praktyce **Pod**-y wykorzystywane są do między innymi monitorowania, usuwania i restartów.

#### Przykład definicji Pod-a
```
apiVersion: v1
kind: Pod
metadataL
	name: my-nginx-pod
	labels:
		app: nginx
spec:
	containers:
	- name: nginx-container
		image: nginx:latest
		ports:
		- containerPort: 80
	- name: sidecar-container
		image: busybox
		command: ['sh', '-c', 'while true;do echo hello; sleep 10; done']
```