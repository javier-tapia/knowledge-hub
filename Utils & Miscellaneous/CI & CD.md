<h1>CI/CD</h1>

***Index***:
<!-- TOC -->
  * [¿Qué es CI/CD?](#qué-es-cicd)
  * [*GitHub Actions*](#github-actions)
    * [La anatomía de una GHA](#la-anatomía-de-una-gha)
    * [Ejemplo de código](#ejemplo-de-código)
<!-- TOC -->

## ¿Qué es CI/CD?
CI/CD es el método para lograr que los cambios hechos en el código se prueben y lleguen a los usuarios de forma automática, sin tener que hacer procesos manuales propensos a errores.  
Se divide en dos partes complementarias:
- **CI (Integración Continua)** :arrow_right: Cada vez que un desarrollador sube código nuevo, un servidor automático lo "descarga", lo compila y **ejecuta pruebas matemáticas (tests)** para asegurarse de que el código nuevo no rompió lo que ya funcionaba.
- **CD (Despliegue Continuo)** :arrow_right: Si todas las pruebas del paso anterior salen bien, el mismo servidor se encarga de **subir el código automáticamente a producción** (a la App Store, a AWS, al servidor web, etc.) para que los usuarios finales vean la actualización de inmediato.

## *GitHub Actions*
[GitHub Actions](https://github.com/features/actions) es la herramienta de CI/CD integrada de forma nativa dentro de GitHub. Sirve para automatizar tareas mediante archivos de configuración en el repositorio.  
Tiene una ventaja enorme: el _Marketplace_. Hay más de 20,000 "_Actions_" listas creadas por la comunidad.

### La anatomía de una GHA
1. ***Workflow* (el *Pipeline* completo)**: Es el archivo YAML (``.github/workflows/main.yml``) donde se define todo el proceso. Es el flujo automatizado completo de principio a fin (ej. "_Compilar, testear y desplegar en AWS_").
2. ***Job* (las Etapas)**: Un _workflow_ se divide en bloques grandes. Por ejemplo, un _Job_ para "Pasar Tests" y otro _Job_ para "Desplegar".
3. ***Step* (los Pasos)**: Dentro de cada _Job_ hay acciones individuales secuenciales.
4. ***Action* (la pieza reutilizable)**: Es el bloque de código ejecutable más pequeño. En lugar de escribir 20 líneas de comandos de consola para conectarte a AWS, se manda a llamar una _GitHub Action_ oficial de AWS que hace todo en una sola línea.

### Ejemplo de código

```yaml
name: Mi Pipeline de CI/CD # <--- Esto es todo el PIPELINE (Workflow)

on: # <--- Acá se definen los triggers del PIPELINE
  push:
    branches:
      - main

jobs:
  pruebas_y_despliegue:
    runs-on: ubuntu-latest
    steps:
      # Paso 1: Descarga el código del repositorio usando una ACTION comunitaria
      - name: Descargar Código
        uses: actions/checkout@v4 # <--- Esto es una GitHub Action

      # Paso 2: Un comando de consola normal (No es una Action, es código propio)
      - name: Ejecutar Tests
        run: npm test 

      # Paso 3: Sube el resultado a internet usando otra ACTION de un tercero
      - name: Desplegar en Firebase
        uses: w9jds/firebase-action@master # <--- Esto es otra GitHub Action
```

