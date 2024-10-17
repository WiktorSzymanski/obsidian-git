---
aliases:
  - ZSR
---
# Dockerfile
---
## Struktura Dockerfile-a
- **FROM** - Definiuje obraz bazowy
- **RUN** - Wykonuje polecenia w czasie budowy obrazu
- **COPY/ADD** - Kopiuje pliki do obrazu
- **CMD/ENTRYPOINT** - Definiuje polecenie uruchamiane przy starcie kontenera

## Optymalizacja
- używaj mniejszych obrazów bazowych
- minimalizuj liczbę warstw w **Dockerfile**
- multi-stage builds

Przy budowie **Dockerfile**-a powinno się go budować warstwami, gdzie na samej górze powinny być najrzacziej zmieniane, podczas gdy niżej te które się zmieniają coraz to częściej