<h1>Crear módulos Gradle</h1>

***Index***:
<!-- TOC -->
  * [Creación manual](#creación-manual)
    * [1. Crear el directorio del módulo](#1-crear-el-directorio-del-módulo)
    * [2. Registrar el módulo en el proyecto](#2-registrar-el-módulo-en-el-proyecto)
    * [3. Crear el *script* de construcción](#3-crear-el-script-de-construcción)
    * [4. Sincronizar y vincular la dependencia](#4-sincronizar-y-vincular-la-dependencia)
  * [Creación desde Android Studio](#creación-desde-android-studio)
    * [Lo que hace el asistente por detrás](#lo-que-hace-el-asistente-por-detrás)
  * [El asunto del ``AndroidManifest.xml``](#el-asunto-del-androidmanifestxml)
    * [¿Cuándo SÍ es obligatorio crear un ``AndroidManifest.xml`` en un módulo?](#cuándo-sí-es-obligatorio-crear-un-androidmanifestxml-en-un-módulo)
<!-- TOC -->

---

## Creación manual
### 1. Crear el directorio del módulo
En la raíz del proyecto, creá una carpeta con el nombre del módulo (por ejemplo, **_my-module_**) y adentro se agrega la estructura estándar: **_src/main/java/com/my/package/_**.

### 2. Registrar el módulo en el proyecto
Abrir el archivo ``settings.gradle.kts`` en la raíz del proyecto e incluir la nueva ruta:

```kotlin
include(":my-module")
```

### 3. Crear el *script* de construcción
Dentro de la carpeta **_my-module/_**, crear un archivo llamado ``build.gradle.kts`` con la configuración base de librería:

```kotlin
plugins {
    alias(libs.plugins.android.library)
    alias(libs.plugins.kotlin.android)
}

android {
    namespace = "com.my.package.mymodule"
    compileSdk = 36

    defaultConfig {
        minSdk = 26
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }
}
```

### 4. Sincronizar y vincular la dependencia
1. Clic en el botón **_Sync Now_** en Android Studio.
2. Para consumir el nuevo módulo desde el módulo **_:app_**, agregarlo en las dependencias de ``app/build.gradle.kts``:

```kotlin
dependencies {
    implementation(project(":my-module"))
}
```

## Creación desde Android Studio
La única diferencia real con la creación manual, es que el asistente visual **_New > Module_** de Android Studio automatiza todos esos pasos manuales en un solo clic a través de una interfaz gráfica.  
A nivel de resultado final en el disco duro y en el proyecto, el resultado es exactamente el mismo.

### Lo que hace el asistente por detrás
Cuando se usa el asistente (**_File > New > New Module..._**), Android Studio ejecuta una plantilla interna que hace exactamente esto:
1. **Genera las carpetas físicamente**: Crea la carpeta del módulo, la estructura ``src/main/java/...`` y el archivo ``AndroidManifest.xml`` base. 
2. **Escribe el ``build.gradle.kts`` del módulo**: Aplica los _plugins_ de Android (``android.library`` o ``android.application``), asigna el ``namespace`` y configura el ``compileSdk`` y ``minSdk`` leyendo la versión por defecto del proyecto. 
3. **Modifica el ``settings.gradle.kts``**: Agrega automáticamente la línea ``include(":nombre-del-modulo")`` en la raíz. 
4. **Dispara el *Gradle Sync***: Ejecuta la sincronización en segundo plano automáticamente para que el proyecto reconozca el nuevo módulo sin que haya que tocar nada.

## El asunto del ``AndroidManifest.xml``
En las versiones modernas de Gradle y Android Gradle Plugin (AGP), **el archivo ``AndroidManifest.xml`` en un módulo de librería ya no es obligatorio** si el módulo solo contiene código Kotlin/Java puro o clases de infraestructura.  
**AGP genera un Manifest sintético en memoria durante la compilación** usando el ``namespace`` que se define en el ``build.gradle.kts``.

### ¿Cuándo SÍ es obligatorio crear un ``AndroidManifest.xml`` en un módulo?
Solo se necesita agregar un Manifest a un submódulo si este define componentes o recursos propios del sistema Android, tales como:

- **Componentes de Android con contexto**: Si el módulo declara su propia ``<activity>``, ``<service>``, ``<receiver>`` o ``<provider>``. 
- **Permisos específicos**: Si el módulo requiere permisos del sistema que se quieren fusionar (**_manifest merger_**) con la app principal (por ejemplo, ``<uses-permission android:name="android.permission.INTERNET" />``). 
- **Recursos de aplicación**: Temas personalizados de UI o metadatos del sistema.
