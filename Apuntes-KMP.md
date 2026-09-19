<h1>Kotlin Multiplatform (KMP) y Compose Multiplatform (CMP)</h1>

***Index***:
<!-- TOC -->
  * [¿Qué son KMP y CMP?](#qué-son-kmp-y-cmp)
    * [Kotlin Multiplatform (KMP)](#kotlin-multiplatform-kmp)
    * [Compose Multiplatform (CMP)](#compose-multiplatform-cmp)
    * [Ventajas Principales](#ventajas-principales)
    * [*Plugins* recomendados](#plugins-recomendados)
  * [Arquitectura de un Proyecto KMP](#arquitectura-de-un-proyecto-kmp)
    * [Estructura Modular Típica](#estructura-modular-típica)
    * [Flujo de Interacción](#flujo-de-interacción)
    * [El Rol de los Módulos de Plataforma](#el-rol-de-los-módulos-de-plataforma)
  * [Conceptos Fundamentales de KMP](#conceptos-fundamentales-de-kmp)
    * [*Targets*: Los destinos de compilación](#targets-los-destinos-de-compilación)
    * [*Source Sets*: Las carpetas con el código](#source-sets-las-carpetas-con-el-código)
    * [La Relación entre *Target* y *Source Set*](#la-relación-entre-target-y-source-set)
    * [El Mecanismo `expect` y `actual`](#el-mecanismo-expect-y-actual)
  * [Gestión de Dependencias](#gestión-de-dependencias)
    * [Dependencias `api` vs. `implementation`](#dependencias-api-vs-implementation)
  * [Recursos nativos en KMP](#recursos-nativos-en-kmp)
    * [Uso de ``Res`` en lugar de ``R``](#uso-de-res-en-lugar-de-r)
    * [*Strings*](#strings)
    * [Íconos e Imágenes](#íconos-e-imágenes)
    * [*Theme* que usa APIs nativas de Android](#theme-que-usa-apis-nativas-de-android)
    * [El patrón de diseño para *Splash Screens* en KMP](#el-patrón-de-diseño-para-splash-screens-en-kmp)
  * [Testing en KMP](#testing-en-kmp)
    * [Tests en el código común (`commonTest`)](#tests-en-el-código-común-commontest)
    * [Tests específicos de plataforma (`androidTest`, `iosTest`, etc.)](#tests-específicos-de-plataforma-androidtest-iostest-etc)
  * [Nota sobre el desarrollo en Windows](#nota-sobre-el-desarrollo-en-windows)
    * [Trabajar en Windows y en Mac](#trabajar-en-windows-y-en-mac)
  * [*Troubleshooting*](#troubleshooting)
    * [*Warning: The Kotlin Hierarchy Template*](#warning-the-kotlin-hierarchy-template)
    * [Error: `No actual for expect` en iOS](#error-no-actual-for-expect-en-ios)
    * [*Warning: Variable de source set "nunca usada"*](#warning-variable-de-source-set-nunca-usada)
  * [Referencias y Recursos](#referencias-y-recursos)
<!-- TOC -->

---

## ¿Qué son KMP y CMP?
### Kotlin Multiplatform (KMP)
Kotlin Multiplatform (KMP) es una funcionalidad del lenguaje Kotlin que permite escribir y reutilizar código a través de múltiples plataformas. Su objetivo principal es compartir la lógica de negocio, el acceso a datos y los modelos desde una única base de código, mientras se mantiene la flexibilidad para escribir código específico de cada plataforma cuando sea necesario para acceder a APIs nativas.

### Compose Multiplatform (CMP)
Compose Multiplatform es un _framework_ de UI declarativo moderno creado por JetBrains, que extiende Jetpack Compose de Android a otras plataformas. Permite que la interfaz de usuario (UI) también se defina una sola vez en el código común y se renderice de forma nativa en cada plataforma de destino, como _desktop_ (Windows, macOS, Linux), Web, Android y iOS.

### Ventajas Principales
- **Reutilización de código**: La lógica de negocio y, en gran medida, la UI, solo necesitan escribirse una vez.
- **Consistencia y Mantenimiento**: Asegura un comportamiento coherente de la aplicación en todas las plataformas y simplifica la corrección de errores y las actualizaciones.
- **Rendimiento Nativo**: El código Kotlin se compila al formato esperado por la plataforma de destino (ej. bytecode de JVM, binario nativo, JavaScript), por lo que no hay una capa de abstracción que penalice el rendimiento.

### *Plugins* recomendados
1. **Kotlin Multiplatform (de JetBrains)**: Es el plugin oficial que le ayuda al IDE a entender mejor los [Source Sets](#source-sets-las-carpetas-con-el-código) compartidos (``commonMain``, ``iosMain``, etc.) y permite cambiar de forma limpia entre las plataformas.
2. **Compose Multiplatform IDE Support (de JetBrains)**: Da el autocompletado específico para los componentes de Compose que se comparten entre Android e iOS, y soporte para previsualizar recursos (como imágenes o strings compartidos).

## Arquitectura de un Proyecto KMP
### Estructura Modular Típica
Un proyecto KMP se organiza comúnmente en los siguientes módulos:

- **``shared``**: Es el corazón del proyecto. Contiene la lógica de negocio, el acceso a datos, los modelos y, si se usa CMP, la UI que se comparte entre todas las plataformas.
- **``androidApp``**: La aplicación para Android. Este módulo consume el código del módulo ``shared`` y proporciona el punto de entrada (``Activity``) y la configuración específica para Android (``AndroidManifest.xml``).
- **``desktopApp``**: La aplicación de escritorio (JVM). De forma similar, consume el módulo ``shared`` y gestiona el lanzamiento de la aplicación de escritorio a través de su función ``main``.
- **``iosApp``**: Un proyecto de Xcode que consume el _framework_ nativo generado por el módulo `shared`. Proporciona el punto de entrada de la aplicación Swift/Objective-C y gestiona el ciclo de vida de la aplicación iOS.
- **``webApp``**: La aplicación web (JS). Consume la salida de JavaScript del módulo ``shared``. Generalmente contiene el archivo `index.html` y los recursos web, y se encarga de iniciar la aplicación en el navegador. Se lo suele llamar también `jsApp`.

### Flujo de Interacción
1. El módulo **``shared``** define las interfaces, la lógica principal y la UI compartida.
2. Los módulos de plataforma (``androidApp``, ``desktopApp``, ``iosApp``, ``webApp``) dependen del módulo ``shared``.
3. Cada aplicación de plataforma inicializa la capa compartida y utiliza sus componentes para construir la aplicación final y manejar las interacciones del usuario.

### El Rol de los Módulos de Plataforma
Aunque `shared` contiene la mayor parte del código (lógica y UI), los módulos específicos de la plataforma son esenciales por varias razones:

- **Punto de Entrada**: Cada plataforma inicia una aplicación de forma diferente. El módulo `shared` se configura como una librería, no como una aplicación ejecutable. Los módulos de plataforma proveen el código de arranque, que es una pequeña capa de código nativo que se encarga de lanzar la UI compartida.
  - En ``androidApp``: Es una ``Activity`` de Android que, en su ``onCreate``, llama a ``setContent`` para mostrar la UI de Compose del módulo ``shared``.
  - En ``iosApp``: Es un ``UIViewController`` en Swift que carga y presenta la UI de Compose Multiplatform.
  - En ``desktopApp``: Es la función ``main`` que crea la ``Window`` y le dice que dibuje la UI del módulo ``shared``.
  - En ``webApp``: Es el archivo ``index.html`` que carga el ``.js`` compilado y un pequeño script que monta la aplicación en un elemento del DOM (como un ``<div>``).
- **Dependencias y SDK's específicos**: Permiten incluir dependencias que solo son relevantes para su plataforma (ej. librerías de Jetpack en Android).
- **Empaquetado**: El proceso para generar el artefacto final es radicalmente diferente en cada plataforma (APK/AAB para Android, JAR/MSI/DMG para _desktop_, un paquete `.app` para iOS, o un conjunto de archivos estáticos HTML/JS/CSS para Web). Los módulos de aplicación se encargan de esta configuración.

## Conceptos Fundamentales de KMP
### *Targets*: Los destinos de compilación
En KMP, un _target_ representa una plataforma específica para la que se compilará el código. Se declaran en el archivo `build.gradle.kts` del módulo `shared`. Ejemplos comunes son `android`, `jvm` (para _desktop_), `iosArm64`, o `js`.

### *Source Sets*: Las carpetas con el código
Un _source set_ es un conjunto de carpetas con código fuente. Al definir un _target_, el _plugin_ de Kotlin crea automáticamente un _source set_ para él.
- **`commonMain`**: Contiene el código que es 100% independiente de la plataforma y se comparte con todos los _targets_.
- **`androidMain`**, **`desktopMain`**, etc.: Contienen la implementación específica para una plataforma concreta.

### La Relación entre *Target* y *Source Set*
> **Los _Targets_ definen el "para qué" (para qué plataformas compilar). Los _Source Sets_ organizan el "dónde" (dónde vive el código para esas plataformas).**

El código en `commonMain` se compila para todos los _targets_. El código en un _source set_ específico (como `androidMain`) solo se compila para el _target_ correspondiente.

### El Mecanismo `expect` y `actual`
Este mecanismo permite que el código común declare una funcionalidad que necesita, dejando que cada plataforma provea su propia implementación.
- En `commonMain`, se define una declaración con la palabra clave `expect` (ej. `expect fun getPlatformName(): String`).
- En cada _source set_ de plataforma (`androidMain`, `desktopMain`), se provee la implementación `actual` correspondiente (ej. `actual fun getPlatformName(): String = "Android"`).

Así, el código común puede llamar a `getPlatformName()` sin conocer la plataforma subyacente.

## Gestión de Dependencias
Las dependencias se gestionan en los archivos `build.gradle.kts`. Dentro del módulo `shared`, es posible declararlas para cada _source set_:
- **`commonMain`**: Para librerías multiplataforma como **_Ktor_** (red), **_SQLDelight_** (base de datos) o **_Kotlinx Serialization_**.
- **`androidMain`**, **`desktopMain`**, etc.: Para dependencias específicas de una plataforma, como motores concretos para una librería o acceso a una API nativa. Por ejemplo, para hacer una petición de red con **_Ktor_**, en Android (``androidMain``) se usaría el motor **_OkHttp_**, pero en la app de escritorio (``desktopMain``) se usaría **_CIO_**.

### Dependencias `api` vs. `implementation`
Cuando se añade una dependencia a un _source set_ específico (ej. a `desktopMain` en el módulo `shared`), por defecto, esa dependencia es privada y no es visible para los módulos que consumen `shared` (como `desktopApp`). Este es un mecanismo de encapsulamiento para evitar que las dependencias de una plataforma se filtren a otra.

Para controlar esta visibilidad, se usan dos tipos de declaraciones:

- **`implementation`**: Es la opción por defecto. La dependencia es un detalle de implementación privado y no se expone a los módulos consumidores. Es ideal para dependencias internas que los módulos de aplicación no necesitan conocer.
  - *Ejemplo*: El motor específico de **_Ktor_** (como **_OkHttp_** o **_CIO_**). El módulo `desktopApp` no necesita saber qué motor se está usando, solo quiere un `HttpClient` funcional.

- **`api`**: La dependencia se convierte en parte de la "Interfaz Pública de Programación" (*API*) del módulo y se exporta a los módulos consumidores. Esto es necesario solo cuando las clases o funciones de la dependencia deben ser usadas directamente en el módulo consumidor.
  - *Ejemplo*: En una aplicación Compose for Desktop, el módulo `desktopApp` necesita usar las funciones `Window()` y `application()` de la librería `compose.desktop.currentOs`. Por lo tanto, el módulo `shared` debe declarar esta dependencia usando `api` para "exportar" esa capacidad.

**Regla general**: Usar siempre `implementation` a menos que sea estrictamente necesario que el módulo consumidor interactúe directamente con las clases de la dependencia. Esto mejora el encapsulamiento y los tiempos de compilación.

## Recursos nativos en KMP
Las clases exclusivas del SDK nativo de Android no son aplicables a todo el entorno compartido de Kotlin Multiplatform. Por lo cual, deben resolverse por separado y meter lógica dentro del módulo ``commonMain`` solo cuando lo que utiliza es agnóstico a la plataforma.

### Uso de ``Res`` en lugar de ``R``
Es una de las diferencias más importantes al pasar de Android nativo a Compose Multiplatform (CMP).  
1. **`R` es solo para Android**  
   El clásico `R.string.texto` o `R.drawable.imagen` es una clase generada por el sistema de construcción de Android (AAPT2). Como esa herramienta no existe en iOS o Desktop, no se puede usar en el código compartido (`commonMain`).
2. **`Res` es Multiplatform**  
   JetBrains creó la librería **Compose Resources** para unificar el acceso a cualquier tipo de recurso en una estructura común.
    - **Ubicación**: Todos los archivos se centralizan en la carpeta obligatoria `shared/src/commonMain/composeResources/drawable/`.
    - **Estructura por tipo**: Se organizan en subcarpetas fijas según su naturaleza:
      - ``drawable/`` -> Imágenes (PNG, JPG, Vector Drawables XML). 
      - ``values/`` -> Textos y traducciones (``strings.xml``). 
      - ``font/`` -> Fuentes tipográficas (TTF, OTF). 
      - ``files/`` -> Archivos genéricos o assets crudos (JSON, TXT, etc.).
    - **Generación y Acceso**: El plugin de Compose genera automáticamente la clase `Res` (en lugar de `R`), permitiendo accesos del tipo ``Res.drawable.mi_foto``, ``Res.string.mi_texto`` o ``Res.font.mi_fuente``.
3. **Es *Type-Safe* (Seguro)**  
   A diferencia de `R` que devuelve un `Int` (un ID numérico propenso a errores), `Res` devuelve objetos con tipado estricto (``DrawableResource``, ``StringResource``, ``FontResource``). Esto evita que se le pase por error un ID de un String a una función que espera una imagen.

### *Strings*
En KMP, las carpetas correspondientes a los recursos de textos se deben crear de forma 100% manual, respetando estrictamente las mayúsculas y minúsculas en las rutas.

```text
shared/
└── src/
    └── commonMain/
        └── composeResources/
            ├── values/            <-- Idioma por defecto / Fallback (Ej.: Español)
            │   └── strings.xml
            └── values-en/         <-- Traducción a Inglés
                └── strings.xml
```

### Íconos e Imágenes
📌 **SVG vs PNG**

- **SVG (Iconos e ilustraciones simples)**:
  - **Ventaja**: Son "infinitos". Se ven perfectos en cualquier resolución (desde un teléfono barato hasta una tablet 4K) sin pixelarse. 
  - **Peso**: El archivo pesa unos pocos KB. 
  - **Uso**: En CMP, los SVG se cargan directamente con `painterResource` y funcionan en iOS/Desktop, **pero rompen la aplicación en Android en tiempo de ejecución con una `IllegalStateException`** porque el sistema operativo Android no soporta el formato SVG crudo nativamente. Para que sea verdaderamente multiplataforma, **el SVG debe convertirse previamente a XML (_VectorDrawable_)** usando el **_Resource Manager_** (o *Vector Asset Studio*) antes de meterlo en `commonMain`.
  - **Compatibilidad**: CMP soporta el formato XML de Android de manera nativa e idéntica en **todas las plataformas** (incluyendo iOS). Por lo tanto, usar `.xml` para vectores en la carpeta compartida garantiza compatibilidad universal y cero crasheos.
- **PNG (Fotos o imágenes con sombras/degradados complejos)**:
  - Usarlos solo si el SVG se ve mal o si es una fotografía real. 
  - **Desventaja**: Habría que exportar varias versiones para que no se vean borrosos en pantallas de alta densidad.

📌 **¿Dónde guardarlos?**

Dentro de la ruta ``shared/src/commonMain/composeResources/drawable/``. El plugin de recursos de Compose busca carpetas específicas para saber qué tipo de recurso es.

📌 **Nomenclatura**

Usar siempre minúsculas y guiones bajos (ej.: ``ic_google_logo.svg``, ``img_welcome_hero.png``). Evitar mayúsculas o espacios, ya que el compilador de recursos fallará.

📌 **Image Asset (Bitmaps) vs. Vector Asset (Vectores)**

Para Android nativo (módulo ``androidApp``), Android Studio dispone de dos herramientas para trabajar con imágenes y/o íconos: **_Image Asset_** y **_Vector Asset_**.  
El **_Vector Asset_** optimiza peso, mantenibilidad de código y escalabilidad multiplataforma, pero se debe usar el **Image Asset** cuando el procesador del dispositivo gastaría más recursos computacionales renderizando matemáticas complejas (como un degradé fotográfico) que simplemente leyendo un mapa de píxeles ya cocinado.

| Criterio                                   | Image Asset (Bitmaps: PNG, JPG, WebP)                                                                                           | Vector Asset (Vectores: SVG, VectorDrawable)                                                                                                                                                                                                                                                                                                                                             |
|:-------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **1. Naturaleza Técnica**                  | Matriz rígida de píxeles (mapa de bits).                                                                                        | Fórmulas matemáticas (nodos, líneas, curvas).                                                                                                                                                                                                                                                                                                                                            |
| **Comportamiento al escalar**              | Se pixela si se agranda; pierde definición si se achica drásticamente.                                                          | **Infinitamente escalable.** Mantiene nitidez perfecta en cualquier resolución.                                                                                                                                                                                                                                                                                                          |
| **Impacto en el almacenamiento**           | Alto. El peso crece exponencialmente con la resolución y cantidad de archivos.                                                  | Ultra bajo. Es texto plano formateado en XML, pesa apenas unos pocos kilobytes.                                                                                                                                                                                                                                                                                                          |
| **2. En Android Nativo (`:androidApp`)**   | Genera de **5 a 7 copias físicas** de la imagen en diferentes densidades (`mdpi` a `xxxhdpi`).                                  | Genera **un único archivo XML** (`VectorDrawable`) que el sistema dibuja en tiempo real. **Nota**: Si se usa para el _Launcher Icon_ vía **_New Image Asset_**, creará el XML en ``anydpi`` junto con carpetas de densidades de respaldo por retrocompatibilidad.                                                                                                                        |
| **Uso en Launcher Icons**                  | **Obligatorio solo si el asset es un mapa de bits**, forzando al asistente a subdividir el PNG en capas de píxeles.             | **Estándar de Oro.** Permite que la física del Launcher anime el ícono usando geometría matemática nativa.                                                                                                                                                                                                                                                                               |
| **3. Entorno Multiplataforma (KMP / CMP)** | Se colocan en `composeResources/drawable` y requieren `painterResource()`. Pueden requerir variantes si se busca alta densidad. | Soporte nativo directo en `commonMain` **SIEMPRE QUE SEA XML**. Un solo archivo **XML (_VectorDrawable_)** sirve para Android, iOS (sin Xcode) y Desktop. Si se deja como `.svg`, Android lanzará un `IllegalStateException`.                                                                                                                                                            |
| **Mejor Caso de Uso**                      | Fotografías reales, ilustraciones hiperrealistas, logos con texturas complejas o degradés 3D avanzados.                         | Iconos de interfaz (UI), botones, logotipos planos, barras de navegación e ilustraciones vectoriales.                                                                                                                                                                                                                                                                                    |

📌 ***Launcher Icons***

Aunque KMP comparte la interfaz interna, los ***launcher icons*** (íconos del escritorio) siguen siendo recursos estrictamente administrados por el sistema operativo nativo. Por lo tanto, **independientemente de si el logo original es un vector o una imagen**, se debe configurar el ícono en cada módulo específico adaptándose a los formatos exigidos por cada plataforma:

- **Android (módulo ``androidApp``)**: Funciona igual que en un proyecto nativo. Se usa obligatoriamente el asistente **New Image Asset** en la carpeta ``androidApp/src/main/res/mipmap``. Si se le carga un SVG, el asistente lo traducirá automáticamente a los archivos XML vectoriales adaptativos requeridos por Android (``ic_launcher.xml``).
- **iOS (módulo ``iosApp``)**: Apple no soporta SVGs para el ícono de la app. Se debe abrir el proyecto en **Xcode**, ir a ``iosApp/iosApp/Assets.xcassets/AppIcon.appiconset``, configurar en **"Single Size"** e importar obligatoriamente un **PNG de 1024x1024 px**. El compilador de Apple procesará y escalará este _asset_ único directamente en el empaquetado final de la aplicación.
- **Desktop (módulo ``desktopApp``)**: Los sistemas operativos de escritorio no usan vectores para sus accesos directos. Se configuran en el ``build.gradle.kts`` dentro del bloque ``compose.desktop { application { nativeDistributions { ... } } }``. **Nota**: Las ``nativeDistributions`` aplican al instalador final (``.exe``, ``.dmg``, ``.deb``). El ícono de la ventana en tiempo de ejecución de desarrollo se asigna por código en la función ``Window(icon = ...)``. Requieren archivos en formatos nativos específicos: ``.ico`` para Windows, ``.icns`` para macOS y ``.png`` para Linux.

```text
📂 proyecto-kmp/
│
├── 📂 androidApp/           <-- MÓDULO DE APLICACIÓN DICTADO POR ANDROID
│   └── 📂 src/
│       └── 📂 main/
│           └── 📂 res/mipmap/ <-- 1. EL ÍCONO DEL ESCRITORIO DE ANDROID VIVE ACÁ
│
├── 📂 iosApp/               <-- MÓDULO DE APLICACIÓN DICTADO POR XCODE
│   └── 📂 iosApp/Assets.xcassets/AppIcon.appiconset/ <-- 3. EL ÍCONO DEL ESCRITORIO DE IOS VIVE ACÁ
│
└── 📂 shared/               <-- EL MÓDULO DE LÓGICA/RECURSOS COMPARTIDOS
    └── 📂 src/
        └── 📂 commonMain/   <-- 2. LAS IMÁGENES DE ADENTRO DE LA UI COMPARTIDA VIVEN ACÁ
            └── 📂 composeResources/drawable/ (Splash, Topbar, etc.)
```

### *Theme* que usa APIs nativas de Android
No sería posible usar este fragmento de código en el archivo ``Theme.kt`` si se alojara dentro del módulo ``commonMain``, puesto que la clase ``Build``, la propiedad ``LocalContext``, y las funciones ``dynamicDarkColorScheme`` y ``dynamicLightColorScheme`` son exclusivas de Android.

```kotlin
dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        }
```

### El patrón de diseño para *Splash Screens* en KMP
> 🔍 Ver también https://developer.android.com/develop/ui/views/launch/splash-screen

La arquitectura para implementar pantallas de bienvenida impecables en KMP sigue siempre este flujo dividido en dos capas:

```text
┌─────────────────────────────────────────────────────────────────────────────────────┐
│ 1. CAPA NATIVA (Milisegundo Cero)                                                   │
├───────────────────────────────────┬─────────────────────────────────────────────────┤
│ Android: androidx.core-splash     │ iOS: Info.plist (UILaunchScreen) / Assets       │
│ Lee el XML nativo del sistema.    │ Lee la interfaz nativa de Apple.                │
└───────────────────────────────────┴─────────────────────────────────────────────────┘
                                 │
                                 ▼ (Compose se inicializa en memoria)
┌────────────────────────────────────────────────────────────────────────┐
│ 2. CAPA COMPARTIDA (commonMain)                                        │
├────────────────────────────────────────────────────────────────────────┤
│ SplashScreen.kt (Composable)                                           │
│ Ej.: Muestra el logo limpio con transparencia, corre el spinner y      │
│  valida la sesión (Ktor/DataStore) en segundo plano antes de navegar.  │
└────────────────────────────────────────────────────────────────────────┘
```

Al aplicar esta estrategia doble, se garantiza una experiencia de usuario fluida, sin saltos de color bruscos (_glitches_ visuales), cumpliendo con los estándares de diseño tanto de la _Google Play Store_ como de la _Apple App Store_.

📌 **Repaso en Android**:
- Implementar la lib ``androidx.core:core-splashscreen``
- Crear los archivos ``androidApp/src/main/res/values/themes.xml`` y su contraparte `androidApp/src/main/res/values-night/themes.xml`
- Asignar el _theme_ en el _Manifest_ (ej.: `android:theme="@style/Theme.App.Starting"`)
- Vincularlo en la _MainActivity_ (``installSplashScreen()`` al principio del `onCreate`)

> 💡 NOTA DE RENDIMIENTO (Buenas prácticas de Google):  
> Evitar crear una pantalla intermedia "SplashScreen.kt" en ``commonMain`` que muestre otro spinner justo después de la splash nativa. Esto rompe la fluidez visual. La documentación de Android recomienda retener la Splash Screen nativa en pantalla usando: ``splashScreen.setKeepOnScreenCondition { !viewModel.isReady }``.  
> De esta forma, la app se queda congelada en el logo del sistema mientras se cargan las configuraciones en segundo plano, y salta directo a la UI final (Home o Login) de forma limpia.

## Testing en KMP
La estrategia de testing en KMP sigue la estructura de los _source sets_ para maximizar la reutilización y, al mismo tiempo, garantizar el correcto funcionamiento en cada plataforma.

### Tests en el código común (`commonTest`)
Este es el lugar ideal para la gran mayoría de los tests unitarios. El código escrito aquí se compila y ejecuta en todas las plataformas _target_.
- **Qué testear aquí**: Lógica de negocio pura (casos de uso, _ViewModels_/_Presenters_), validaciones, _mappers_ de datos, y cualquier componente que no tenga dependencias de APIs de plataforma.
- **Ventaja principal**: Se escriben una sola vez y verifican que el comportamiento central de la aplicación sea consistente en todos los _targets_.
- **Limitación**: No se puede acceder a APIs específicas de una plataforma (como el `Context` de Android o `NSUserDefaults` de iOS).

### Tests específicos de plataforma (`androidTest`, `iosTest`, etc.)
Estos _source sets_ se utilizan para testear código que sí depende de una plataforma concreta.

**Qué testear aquí**:
  - Las implementaciones `actual` de las declaraciones `expect`. Por ejemplo, si se tiene una `expect` para leer/escribir en disco, aquí se testea que la implementación `actual` para Android use `SharedPreferences` correctamente y la de iOS use `NSUserDefaults`.
  - El código que interactúa directamente con los SDKs nativos.
  - Tests de instrumentación o UI que necesitan un emulador, simulador o dispositivo físico para ejecutarse. Por ejemplo, los tests de UI de Compose que verifican que un `@Composable` se renderiza correctamente en una pantalla de Android.

## Nota sobre el desarrollo en Windows
**UPDATE**: Para crear un proyecto nuevo, es más simple y efectivo usar el [*wizard* oficial de JetBrains](https://kmp.jetbrains.com/?android=true&ios=true&iosui=compose&includeTests=true), el cual genera un .zip limpio con las últimas dependencias estables y una estructura unificada.

En Windows, Android Studio no ofrece directamente algunas plantillas para crear un proyecto KMP desde cero. Esto no impide usar la tecnología, pero implica que la configuración inicial puede requerir más pasos manuales, como:
- Crear manualmente la estructura de módulos (`shared`, `androidApp`, etc.).
- Configurar los archivos de Gradle y las dependencias.

El proyecto resultante, sin embargo, funciona de la misma manera que uno creado en Mac o Linux.

### Trabajar en Windows y en Mac
**Las únicas dos herramientas que se necesitan en Mac**:
1. **Android Studio (con el plugin Kotlin Multiplatform):**
   - Se usa exactamente igual que en Windows. 
   - Se modifica el código común (`commonMain`) y el de Android.
   - **La ventaja en Mac:** Habilitará un botón de "Play" para ejecutar la app directamente en el simulador de iPhone sin salir de Android Studio.
2. **Xcode (el IDE oficial de Apple):**
   - Se descarga gratis desde la Mac App Store.
   - **¿Cuándo es obligatorio abrir Xcode?** :arrow_right: Únicamente para importar los íconos nativos de iOS (`AppIcon`), configurar la pantalla de carga/inicio, estionar los **permisos del dispositivo** (como cámara, ubicación o notificaciones) en el archivo `Info.plist`, configurar los **certificados de desarrollo** y perfiles de aprovisionamiento (Provisioning Profiles) para poder subir la app a la App Store o probarla en un iPhone físico y resolver errores complejos de enlazado de librerías de terceros (CocoaPods o Swift Package Manager). Una vez configurado eso, ya se puede cerrar.

**¿Cómo interactúan entre sí?**:  
Cuando se está en Android Studio en la Mac y se le da al botón de ejecutar en iOS, ocurre lo siguiente por detrás:
- Android Studio compila el código Kotlin mediante Gradle.
- Gradle genera un archivo llamado `Framework` (el código que entiende iOS).
- Android Studio llama de forma invisible a las herramientas ocultas de **Xcode** para que agarren ese `Framework`, lo junten con la carpeta de iOS y levanten el simulador de iPhone.

## *Troubleshooting*
### *Warning: The Kotlin Hierarchy Template*
**El _warning_**:  
> The Default Kotlin Hierarchy Template was not applied to 'project ':shared'':
> Explicit .dependsOn() edges were configured for the following source sets: [desktopMain]
> Consider removing dependsOn-calls or disabling the default template...

**¿Qué significa?**  
En las versiones modernas de KMP, Gradle intenta crear automáticamente una "jerarquía de _source sets_". Por ejemplo, sabe que `androidMain` y `desktopMain` son plataformas JVM y podría crear un _source set_ intermedio `jvmMain` del que ambos dependan para reducir duplicación de código.
El _warning_ aparece porque en el archivo `build.gradle.kts` del módulo `shared` existe una configuración manual como esta:
```kotlin
val desktopMain by getting {
    dependsOn(commonMain) // Esta línea causa el warning
}
```
Al escribir manualmente `dependsOn(commonMain)`, se deshabilita la plantilla de jerarquía (_hierarchy template_) automática. Gradle simplemente está informando que el usuario tomó el control manual.

**¿Es un problema?**  
No, para la mayoría de los proyectos simples no es un problema. Una configuración manual es perfectamente clara y correcta. La plantilla automática es más útil en proyectos masivos con muchos _targets_.

**¿Cómo solucionarlo?**  
Existen dos opciones:
1.  **(Recomendado) Silenciar el _warning_**: Si la configuración manual es la deseada, simplemente se le indica a Gradle que no advierta sobre ello.
    -   Abrir el archivo `gradle.properties`.
    -   Añadir la siguiente línea:
        ```groovy
        kotlin.mpp.applyDefaultHierarchyTemplate=false
        ```
2.  **(Alternativa) Usar la plantilla**: Eliminar las llamadas `dependsOn(commonMain)` de los _source sets_ (`desktopMain`, `androidMain`, etc.). La plantilla por defecto los conectará automáticamente a `commonMain`. Esta es la forma considerada "moderna".

### Error: `No actual for expect` en iOS
**El error**:  
> No actual for expect declaration in module(s): iosSimulatorArm64Main, iosX64Main, iosArm64Main

**¿Qué significa?**  
Este error ocurre porque una declaración `expect` en `commonMain` no tiene su correspondiente implementación `actual` en los _targets_ finales de iOS.  
La razón principal es la diversidad de arquitecturas de _hardware_ en el ecosistema de Apple:
- **Android y _Desktop_ (JVM)**: Al compilar para estas plataformas, el resultado es _bytecode_ de la JVM, que es universal. Una Máquina Virtual de Java (como ART en Android) se encarga de interpretarlo para la arquitectura específica del dispositivo (ARM, x86, etc.). Por ello, solo se necesita un único _target_.
- **iOS (Kotlin/Native)**: Kotlin/Native compila el código Kotlin directamente a código máquina nativo, sin una máquina virtual intermedia. Esto obliga a generar un binario diferente para cada arquitectura de procesador que se quiera soportar:
    - `iosArm64`: Para iPhones y iPads físicos modernos (procesadores Apple Silicon).
    - `iosX64`: Para simuladores de iOS en Macs con procesadores Intel.
    - `iosSimulatorArm64`: Para simuladores de iOS en Macs con procesadores Apple Silicon (M1, M2, etc.).

La parte sutil es que, aunque se haya colocado la implementación `actual` en un _source set_ común para iOS como `iosMain`, Gradle no sabe automáticamente que estos _targets_ específicos deben usar ese código.

**¿Es un problema?**  
Sí, es un error de compilación que impide generar el _framework_ para iOS. El compilador necesita saber exactamente dónde encontrar la implementación `actual` para cada _target_ final.

**¿Cómo solucionarlo?**  
La solución es configurar explícitamente en el archivo `build.gradle.kts` del módulo `shared` que los _source sets_ de los _targets_ específicos de iOS dependen de `iosMain`.

```kotlin
// Se obtiene una referencia al source set iosMain (o se crea si no existe)
val iosMain by creating {
    // Se vincula explícitamente con commonMain
    dependsOn(commonMain)
}

// Se vincula cada target específico de iOS con iosMain
val iosX64Main by getting {
    dependsOn(iosMain)
}
val iosArm64Main by getting {
    dependsOn(iosMain)
}
val iosSimulatorArm64Main by getting {
    dependsOn(iosMain)
}
```

### *Warning: Variable de source set "nunca usada"*
**El _warning_**  
El IDE marca una variable de _source set_ como `androidMain` como si nunca fuera usada.

**¿Qué significa?**  
Es una peculiaridad del editor de código de Android Studio y el DSL de Gradle que se puede ignorar de forma segura.

- **Lo que realmente pasa**: Aunque parece que la variable `androidMain` no se usa, en realidad se está utilizando para configurar el _source set_ a través de la función `by getting`. El _plugin_ de Kotlin Multiplatform utiliza internamente esta declaración para localizar las carpetas (ej. `src/androidMain/kotlin`), configurar dependencias y conectar todo el grafo de compilación.
- **Por qué el IDE se confunde**: El análisis estático del IDE no siempre es lo suficientemente inteligente como para entender que estas declaraciones `val` en el DSL de Gradle son, de hecho, "usadas" por el sistema de _build_ en segundo plano. Simplemente ve una variable a la que no se hace referencia explícita y la marca como no utilizada.

**En resumen**: Es un falso positivo del IDE. El código es correcto y necesario.

## Referencias y Recursos
- [Documentación oficial de Kotlin Multiplatform](https://kotlinlang.org/docs/multiplatform-get-started.html)
- [Documentación oficial de Compose Multiplatform](https://www.jetbrains.com/lp/compose-multiplatform/)
- [Proyecto de práctica - KmpClientSample](https://github.com/javier-tapia/KmpClientSample)
