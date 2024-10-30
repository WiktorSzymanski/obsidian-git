---
aliases:
  - ZSR
---
# Dockerfile
---
## Struktura Dockerfile-a
- **FROM** - Definiuje obraz bazowy
- **MAINTAINER** - Użytkownik zarządzający obrazem
- **RUN** - Wykonuje polecenia w czasie budowy obrazu
- **COPY/ADD** - Kopiuje pliki do obrazu
- **CMD/ENTRYPOINT** - Definiuje polecenie uruchamiane przy starcie kontenera
#### Przykład
```Dockerfile
FROM apline:latest
MAINTAINER voytek
RUN apk update && \
	apk upgrade && \
	apk add nginx && \
	rm -rf /var/cache/apk/*
#TODO
EXPOSE 80
ENV PATH $PATH:/data/bin
VOLUME /data
```
## Optymalizacja
- używaj mniejszych obrazów bazowych
- minimalizuj liczbę warstw w **Dockerfile**
- multi-stage builds

Przy budowie **Dockerfile**-a powinno się go budować warstwami, gdzie na samej górze powinny być najrzacziej zmieniane, podczas gdy niżej te które się zmieniają coraz to częściej