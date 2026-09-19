<h1>Jetpack Compose</h1>

[Jetpack Compose](https://developer.android.com/compose) es el _toolkit_ moderno de Android para construir interfaces de usuario de forma **declarativa**. En lugar de manipular vistas imperativamente, se describe **cómo debería verse la UI según su estado**, y Compose se encarga de renderizarla y actualizarla automáticamente.

> 🔍 Ver también el proyecto [android-and-kotlin-lab-chronicles](https://github.com/javier-tapia/android-and-kotlin-lab-chronicles/tree/master/app/src/main/java/com/example/android_and_kotlin_lab_chronicles/experiments/jetpack_compose)

***Index***:
<!-- TOC -->
  * [Algunos conceptos básicos](#algunos-conceptos-básicos)
    * [*UI State* vs *UI Events*](#ui-state-vs-ui-events)
    * [*State hoisting*](#state-hoisting)
    * [``Modifier``](#modifier)
    * [*Slot API* & ``Scaffold``](#slot-api--scaffold)
    * [*Theme*: Definición de colores, formas y tipografías](#theme-definición-de-colores-formas-y-tipografías)
      * [``Color.kt``](#colorkt)
      * [``Shape.kt``](#shapekt)
      * [``Type.kt``](#typekt)
      * [``Theme.kt``](#themekt)
    * [Estructura del código](#estructura-del-código)
  * [Control del ciclo de composición](#control-del-ciclo-de-composición)
    * [Recomposición & Estabilidad](#recomposición--estabilidad)
    * [La función `key`](#la-función-key)
  * [El estado en *Compose*](#el-estado-en-compose)
    * [`remember` & `rememberSaveable`](#remember--remembersaveable)
    * [Compose ``State``](#compose-state)
      * [Primero: qué es `snapshot`](#primero-qué-es-snapshot)
      * [La función `mutableStateOf`](#la-función-mutablestateof)
      * [La función `produceState`](#la-función-producestate)
      * [La función `derivedStateOf`](#la-función-derivedstateof)
    * [*Flow*](#flow)
      * [La función `collectAsState()`](#la-función-collectasstate)
      * [La función `collectAsStateWithLifecycle()`](#la-función-collectasstatewithlifecycle)
      * [La función `asStateFlow()`](#la-función-asstateflow)
    * [*LiveData*](#livedata)
      * [La función `observeAsState()`](#la-función-observeasstate)
  * [*Side Effects*](#side-effects)
    * [`LaunchedEffect`](#launchedeffect)
    * [`SideEffect`](#sideeffect)
    * [`DisposableEffect`](#disposableeffect)
    * [`rememberCoroutineScope`](#remembercoroutinescope)
    * [`rememberUpdatedState`](#rememberupdatedstate)
    * [`produceState`](#producestate)
    * [`derivedStateOf`](#derivedstateof)
    * [`snapshotFlow`](#snapshotflow)
  * [Algunas comparativas útiles](#algunas-comparativas-útiles)
    * [``MutableState`` vs ``StateFlow``](#mutablestate-vs-stateflow)
    * [`remember` con `key` vs `remember` junto con `derivedStateOf`](#remember-con-key-vs-remember-junto-con-derivedstateof)
    * [`LaunchedEffect` vs `SideEffect`](#launchedeffect-vs-sideeffect)
  * [Animaciones en *Compose*](#animaciones-en-compose)
    * [Qué es *Tween*](#qué-es-tween)
    * [Animaciones *as state*](#animaciones-as-state)
    * [Animaciones de visibilidad](#animaciones-de-visibilidad)
    * [Animaciones de cambio de componentes (*crossfade*)](#animaciones-de-cambio-de-componentes-crossfade)
    * [Animaciones de contenido](#animaciones-de-contenido)
    * [*InfiniteTransition*](#infinitetransition)
  * [Previews](#previews)
    * [Live Template :arrow_right: `prevCol`](#live-template-arrow_right-prevcol)
    * [Usando `PreviewParameterProvider`](#usando-previewparameterprovider)
<!-- TOC -->

---

## Algunos conceptos básicos
### *UI State* vs *UI Events*

- ***UI State*** :arrow_right: **Qué se muestra** en la pantalla y **cómo se muestra** (propiedades intrínsecas a los elementos de la UI que influyen en cómo se renderizan, como el tamaño de fuente, el color, etc.)
- ***UI Events*** :arrow_right: **Acciones** (del usuario o del sistema) que deberían gestionarse en la capa de UI (se incluye al *ViewModel*)

### *State hoisting*

Los estados no deberían estar en los ``Composables`` (deberían ser ***stateless*** en la medida de lo posible, en lugar de ***stateful***).

Para eso, se usa un patrón llamado ***State Hoisting***, que consiste en **_extraer los estados de los ``Composables`` y que el control del mismo recaiga en un miembro de jerarquía superior_**, lo cual permite **_reutilizarlo en otros componentes_**. En vez de crear el estado dentro del `Composable`, se sustituye por dos argumentos: **_uno proporciona el valor_** y el **_otro es una lambda que modifica ese valor_**.

### ``Modifier``

- Permite añadir apariencia o capacidades extra a un composable.
- Es un `companion object` de una interfaz con el mismo nombre.
- Tiene una API de tipo ***Builder***.
- Es importante el orden: **_se van a aplicar en el orden que se hayan definido_**.

Hay varios tipos de modificadores:
- **De posicionamiento y tamaño**: Indica cómo se va a posicionar una vista respecto a las vistas con las que interactúa, así como el espacio que ocuparán en pantalla. Ejemplos: `fillMaxWidth`, `width`, `height`.
- **De funcionalidad**: Permiten ampliar características sobre el composable en el que se aplican. Ejemplos: `clickable`, `horizontalScroll`, `draggable`.
- **De apariencia**: Ejemplos: `background`, `padding` (en Compose, solo existe el `padding`, no hay `margins`), `scale`, `border`, `alpha`.
- **_Listeners_**: Permiten escuchar ciertos eventos relacionados con la vista. Ejemplos: `onFocusChanged`, `onKeyEvent`, `onSizeChanged`.

### *Slot API* & ``Scaffold``

Al igual que en los *xml*, la *AppBar* se puede colocar en cualquier parte de la pantalla. Pero si se quiere usar en la parte superior de la pantalla (lo habitual), lo ideal es usar el composable `Scaffold`. Este componente permite posicionar elementos típicos de *Material* en sus posiciones habituales sin necesidad de hacer nada extra.  
`Scaffold` es el ejemplo perfecto de un patrón que se repite en _Jetpack Compose_, llamado ***Slot API*** (*slot* = `@Composable () -> Unit`). Este patrón consiste básicamente en que **el componente ofrece huecos o *slots*** donde se puede añadir lo que uno quiera (*lambdas* genéricas que aceptan contenido composable). Ver [Practical Compose Slot API example](https://www.valueof.io/blog/compose-slot-api-example-composable-content-lambda)

Para agregar una ``TopAppBar`` en un ``Scaffold``, lo primero que se debe hacer, es ir al *Manifest* y asegurarse que el ***theme*** de la *activity* no tenga (o herede de un tema que no tenga) ``ActionBar``.  
Por ejemplo:

```xml
<style name="Theme.MyMovies" parent="android:Theme.Material.Light.NoActionBar">
```

Por ejemplo, el parámetro `title` de la `TopAppBar` no obliga a que deba contener un `Text` sí o sí. Bien podría contener una `Row` con un texto, un `Spacer` y un ícono:

```kotlin
Scaffold(
    topBar = {
        TopAppBar(title = {
            Row {
                Text(text = stringResource(id = R.string.app_name))
                Spacer(modifier = Modifier.width(16.dp))
                Icon(imageVector = Icons.Default.Android, contentDescription = null)
            }
        })
    }
)
```  

También se puede agregar un **ícono de navegación** de forma muy simple con un `IconButton` (un botón que contiene un ícono):

```kotlin
Scaffold(
    topBar = {
        TopAppBar(
            title = { Text(text = stringResource(id = R.string.app_name)) },
            navigationIcon = {
                IconButton(onClick = { /*TODO*/ }) {
                    Icon(
                        imageVector = Icons.Default.Menu,
                        contentDescription = null
                    )
                }
            }
        )
    }
)
```  

También es posible agregar **acciones de menú**:

```kotlin
Scaffold(
    topBar = {
        TopAppBar(
            title = { Text(text = stringResource(id = R.string.app_name)) },
            actions = {
                IconButton(onClick = { /*TODO*/ }) {
                    Icon(
                        imageVector = Icons.Default.Search,
                        contentDescription = null
                    )
                }
                IconButton(onClick = { /*TODO*/ }) {
                    Icon(
                        imageVector = Icons.Default.Share,
                        contentDescription = null
                    )
                }
            }
        )
    }
)
```

Dentro de esos _slots_, el ``Scaffold`` también puede alojar una ``bottomBar``, un ``snackbarHost`` y varios componentes más. Entre ellos, el ``content`` recibe lo que la app va a pintar como contenido justamente.  
Dicho ``content`` recibe un objeto ``PaddingValues``, el cual refleja el espacio ocupado por la ``topBar``, ``bottomBar`` u otras barras del ``Scaffold``. Si no se define ninguna barra, los valores de _padding_ serán ``0.dp``.

```kotlin
Scaffold(
    snackbarHost = { SnackbarHost(snackbarHostState) },
    topBar = { MyTopAppBar() },
    bottomBar = { MyBottomNavigation() },
    content = { paddingValues ->
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues)
                .consumeWindowInsets(paddingValues)
        ) {
            Text(text = "Text 1")
            Text(text = "Text 2")
        }
    }
)
```

### *Theme*: Definición de colores, formas y tipografías
Al crear un nuevo proyecto en Compose (llamado ``MyApp`` para el ejemplo), se crean automáticamente tres archivos dentro de ``app/src/main/java/com/example/myapp/ui/theme``, que son ``Color.kt``, ``Theme.kt`` y ``Type.kt``.  
Estos archivos conforman el **sistema de _theming_ de Material 3 en Compose**, que separa claramente **colores**, **formas** y **tipografías**, y centraliza todo en un único `MyAppTheme` dentro del archivo ``Theme.kt``. También se pueden crear otros archivos para definir otras características del _theme_, como puede ser el archivo ``Shape.kt``.  
Dicho _theme_ se puede aplicar a toda la app:

```kotlin
MyAppTheme {
    AppContent()
}
```

#### ``Color.kt``
Define y expone la **paleta de colores** que son consumidos por el archivo ``Theme.kt``.  
En Compose + Material 3, esto suele incluir:

- **LightColorScheme** :arrow_right: Colores usados en modo claro.
- **DarkColorScheme** :arrow_right: Colores usados en modo oscuro.
- Definiciones de colores personalizados con `Color(0xFFxxxxxx)`.

```kotlin
val Purple80 = Color(0xFFD0BCFF)
val PurpleGrey80 = Color(0xFFCCC2DC)
val Pink80 = Color(0xFFEFB8C8)

val Purple40 = Color(0xFF6650a4)
val PurpleGrey40 = Color(0xFF625b71)
val Pink40 = Color(0xFF7D5260)
```

#### ``Shape.kt``
Define las **formas globales** que seguirá la app: radios, esquinas redondeadas y estilos de contenedores.  
Material 3 usa estas formas en componentes como botones, tarjetas, diálogos, etc.

```kotlin
val shapes = Shapes(
    extraSmall = RoundedCornerShape(4.dp),
    small = RoundedCornerShape(8.dp),
    medium = RoundedCornerShape(16.dp),
    large = RoundedCornerShape(24.dp),
    extraLarge = RoundedCornerShape(48.dp)
)
```

#### ``Type.kt``
Define la **tipografía global** de la app, es decir, los estilos de texto que usarán todos los componentes de Material 3: títulos, _body_, _labels_, _displays_, etc. De esta forma, toda la tipografía de la app se vuelve consistente y centralizada dentro del sistema de _theming_.  
Material 3 utiliza una estructura llamada `Typography` que agrupa todos estos estilos y permite personalizar:

- **Fuente** :arrow_right: `FontFamily`
- **Peso** :arrow_right: `FontWeight`
- **Tamaño** :arrow_right: `fontSize`
- **Interlineado** :arrow_right: `lineHeight`
- **Tracking** (ajuste uniforme del espacio entre todas las letras de un texto) :arrow_right: `letterSpacing`

En un proyecto recién creado, el archivo puede verse así:

```kotlin
val Typography = Typography(
    labelMedium = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 11.sp,
        lineHeight = 16.sp,
        letterSpacing = 0.5.sp
    )
)
```

Todos estos estilos se consumen automáticamente al usar componentes de Material:

```kotlin
Text("Some text", style = MaterialTheme.typography.labelMedium)
```

También es el lugar donde se agregan **fuentes personalizadas**. Por ejemplo, asumiendo que se quiere agregar una nueva fuente llamada ``tirra``, se debe hacer lo siguiente:

1. Entrar al sitio de [Google Fonts](https://fonts.google.com/).
2. Elegir la **familia de la fuente** (**_Font Family_**) ``tirra`` y hacer click para entrar al detalle (acá también permite previsualizar los distintos estilos en caso de que tenga varios).
3. Hacer click en el botón **_Get Font_** y luego descargarla (usualmente, se guarda un archivo ``.zip`` que contiene un archivo ``.txt`` con la licencia y otros ``.ttf`` con los diferentes estilos de la fuente).
4. Crear un nuevo directorio dentro de ``/res`` llamado ``/font`` y mover el archivo (o los archivos, en caso de que sean varios estilos) ``.ttf`` dentro (se debe cambiar el nombre original de ser necesario, ya que no puede tener espacios ni mayúsculas).
5. Se crea una nueva propiedad de tipo ``FontFamily`` que recibe uno o más objetos de tipo ``Font``, los cuales esperan el ``resId`` (la ubicación de los archivos ``.ttf`` importados en el paso previo) y el ``weight`` (el cual debe corresponderse con el tipo de fuente especificada por el archivo ``.ttf``).

```kotlin
val tirra = FontFamily(
    Font(resId = R.font.tirra_regular, weight = FontWeight.Normal),
    Font(resId = R.font.tirra_black, weight = FontWeight.Black),
    Font(resId = R.font.tirra_bold, weight = FontWeight.Bold),
    Font(resId = R.font.tirra_extra_bold, weight = FontWeight.ExtraBold),
    Font(resId = R.font.tirra_medium, weight = FontWeight.Medium),
    Font(resId = R.font.tirra_semi_bold, weight = FontWeight.SemiBold)
)
```

6. Finalmente, se puede aplicar esa nueva fuente en donde se quiera.

```kotlin
val Typography = Typography(
    labelMedium = TextStyle(
        fontFamily = FontFamily.Default,
        fontWeight = FontWeight.Medium,
        fontSize = 11.sp,
        lineHeight = 16.sp,
        letterSpacing = 0.5.sp
    ),
    bodyLarge = TextStyle(
        fontFamily = tirra,
        fontWeight = FontWeight.SemiBold,
        fontSize = 16.sp,
        lineHeight = 24.sp,
        letterSpacing = 0.5.sp
    ),
    bodyMedium = TextStyle(
        fontFamily = tirra,
        fontWeight = FontWeight.Medium,
        fontSize = 14.sp,
        lineHeight = 20.sp,
        letterSpacing = 2.sp
    ),
    bodySmall = TextStyle(
        fontFamily = tirra,
        fontWeight = FontWeight.Normal,
        fontSize = 12.sp,
        lineHeight = 16.sp,
        letterSpacing = 2.sp
    )
)
```

#### ``Theme.kt``
Es el **archivo central**, responsable de:

1. Elegir entre _light_ y _dark theme_ (según configuración del sistema).
2. Proveer:
   - ``colorScheme``
   - ``typography``
   - ``shapes``
3. Aplicar el tema a toda la UI:

```kotlin
@Composable
fun MyAppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    // Dynamic color is available on Android 12+
    dynamicColor: Boolean = false,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        }

        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        shapes = shapes,
        content = content
    )
}
```

### Estructura del código

Algunas recomendaciones a tener en cuenta a la hora de estructurar el código en *Jetpack Compose*:
- **Crear un ``Composable`` con la base de la aplicación** :arrow_right: Lo ideal es tener un ``Composable`` que defina las configuraciones y reutilizarlo en todos lados. Es importante recordar que **las funciones `Composable` deben devolver `Unit`, ya que estas funciones emiten componentes de UI**, pero no devuelven nada.
- **Dividir el ``Composable`` en otros más pequeños** :arrow_right: Los ``Composables`` deberían ser autoexplicativos y tener nombres semánticos.
- **Crear ``Composables`` que definan las pantallas** :arrow_right: Lo ideal, es crear ``Composables`` para definir cada una de las pantallas.
- **Estructurar los paquetes de UI por pantallas** :arrow_right: El código queda más ordenado, puede crecer de forma más extensible y se podrán crear tanto *features* como ``Composables`` sin que se vuelva un desorden.
- **Extraer las dimensiones** :arrow_right: *Hardcodear* las dimensiones va a representar un problema si se quiere **configurar la aplicación para distintos tamaños de pantalla (dispositivos diferentes)**. Para eso, es preferible extraerlos al archivo ***dimens*** y obtenerlos con el método ``dimensionResource``.

## Control del ciclo de composición

> 🔍 Referencia:  
> https://developer.android.com/develop/ui/compose/lifecycle

### Recomposición & Estabilidad
La **recomposición** es el proceso mediante el cual Compose **vuelve a ejecutar funciones composables** para actualizar la UI cuando un estado cambia.  
No toda la UI se vuelve a pintar: Compose es **inteligente** y solo recompondrá las partes que **dependen del estado modificado**.

Un objeto o función se considera **estable** cuando Compose puede confiar en que:
- No cambiará su **_identidad_** de manera inesperada.
- Sus lecturas de estado pueden detectarse eficientemente.
- Su comparación (`equals`) es confiable para determinar si debe recomponerse.

La estabilidad permite a Compose:
- Evitar recomposiciones innecesarias.
- Memorizar composables sin invalidar la _cache_.
- Optimizar la ejecución enviando menos trabajo al *runtime*.

**En resumen**:
> 🔄 La **_recomposición_** vuelve a pintar la UI cuando el estado cambia.  
> 🧱 La **_estabilidad_** evita recomposiciones cuando el estado no ha cambiado.

### La función [`key`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#key(kotlin.Array,kotlin.Function0))

```kotlin
@Composable
inline fun <T : Any?> key(vararg keys: Any?, block: @Composable () -> T): T
```

Permite **asignar una clave (_key_) estable** a un bloque de composición para **_controlar su identidad_**. Esa _key_ le indica a Compose **cómo identificar ese bloque entre recomposiciones**, lo cual determina cómo se **preserva (o descarta) el estado** interno de los composables dentro del bloque.

Puntos clave de su utilización:
- Para cualquier caso :arrow_right: Si la _key_ cambia, Compose se ve obligado a **_descartar y recrear ese bloque de UI_**, invalidando su estado previo.
- Para **estructuras dinámicas** (listas, bucles, ramas `if/else` o `when`) :arrow_right: Permite que Compose **preserve correctamente el estado de cada elemento**, evitando que confunda un composable con otro cuando cambia el **orden**, la **cantidad** o los **valores** (sin `key`, Compose asigna el estado **_por posición_**, NO por contenido).

Funcionamiento:
- Todos los composables dentro de `key(...) { ... }` se agrupan bajo **_una identidad única_**. Esa clave **_solo necesita ser única entre los composables hermanos generados en el mismo nivel_**, no en toda la pantalla ni globalmente.
- Si la _key_ **_permanece igual_**, Compose **_reutiliza el estado_** asociado a ese bloque.
- Si la _key_ **_cambia_**, Compose **_invalida ese estado_** y **_reconstruye todo el contenido desde cero_**.

Ejemplo:

```kotlin
sealed interface Screen {
    data object Home : Screen
    data object Profile : Screen
    data object Settings : Screen
}

@Composable
fun HomeScreen() {
    var counter by remember { mutableStateOf(0) }
    Button(onClick = { counter++ }) {
        Text("Home: $counter")
    }
}

@Composable
fun ProfileScreen() {
    var name by remember { mutableStateOf("") }
    TextField(value = name, onValueChange = { name = it })
}

@Composable
fun SettingsScreen() {
    var darkMode by remember { mutableStateOf(false) }
    Switch(checked = darkMode, onCheckedChange = { darkMode = it })
}

@Composable
fun ScreenContent(selected: Screen) {
    Column(Modifier.fillMaxSize()) {

        // Controles para cambiar la pantalla
        Row(Modifier.padding(16.dp)) {
            Button(onClick = { /* cambiar a Home */ }) { Text("Home") }
            Button(onClick = { /* cambiar a Profile */ }) { Text("Profile") }
            Button(onClick = { /* cambiar a Settings */ }) { Text("Settings") }
        }

        // --- PUNTO CRÍTICO: Cambio de pantallas ---

        // ❌ Sin key(selected::class)
        //    Compose puede intentar REUSAR el estado entre pantallas.
        //    Ejemplos de problemas:
        //    - El contador de Home aparece en Profile.
        //    - El texto de Profile aparece cuando se va a Settings.
        //    - El estado de Switch en Settings se mantiene al volver a Profile.
        //
        //    Esto ocurre porque Compose ve solo un "bloque" en la composición
        //    y no sabe que debe descartar el estado al cambiar de pantalla.

        // ✅ Con key(selected::class)
        //    - Cada pantalla tiene una identidad estable distinta.
        //    - Cambiar de pantalla invalida el estado previo y crea uno nuevo.
        //    - Home, Profile y Settings conservan su propio estado sin "contaminarse" entre sí.
        //    - Es la forma correcta de manejar pantallas dinámicas sin navigation-compose.

        key(selected::class) {   // <---- La clave identifica qué pantalla es
            when (selected) {
                Screen.Home -> HomeScreen()
                Screen.Profile -> ProfileScreen()
                Screen.Settings -> SettingsScreen()
            }
        }
    }
}
```

## El estado en *Compose*

> 🔍 Referencia:  
> https://developer.android.com/develop/ui/compose/state

### `remember` & `rememberSaveable`

```kotlin
@Composable
inline fun <T : Any?> remember(
    vararg keys: Any?,
    crossinline calculation: @DisallowComposableCalls () -> T
): T

@Composable
fun <T : Any> rememberSaveable(
    vararg inputs: Any?,
    saver: Saver<T, Any> = autoSaver(),
    key: String? = null,
    init: () -> T
): T
```

Las funciones composables pueden usar la API [`remember`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#remember(kotlin.Function0)) para **_almacenar un objeto en memoria mientras el composable permanezca en la composición_**. Puede almacenar tanto objetos mutables como inmutables. Y el valor recordado puede usarse como **_parámetro de otros composables_** o incluso como parte de la lógica que determina **_qué UI se muestra_**.

Se usa frecuentemente junto con `MutableState` (ver sección previa sobre estados). En términos generales, `remember` recibe una _lambda_ `calculation` por parámetro. La primera vez que se ejecuta, invoca esa _lambda_ y almacena su resultado. Y durante recomposiciones posteriores, devuelve el valor almacenado previamente.

`remember` conserva el valor hasta que el composable sale de la composición. Sin embargo, existe una forma de invalidar ese valor en _cache_, ya que también acepta una o varias **_keys_** por parámetro. ***Si cualquiera de esos keys cambia, en la próxima recomposición `remember` invalidará el valor almacenado en cache y volverá a ejecutar la lambda de `calculation`.***

Ejemplo:

```kotlin
var isVisible by remember { mutableStateOf(true) }

Column(Modifier.fillMaxSize()) {
    Button(onClick = { isVisible = !isVisible }) {
        _root_ide_package_.org.w3c.dom.Text("Show/Hide")
    }

    Spacer(modifier = Modifier.size(50.dp))

    AnimatedVisibility(
        isVisible,
        enter = slideInHorizontally(),
        exit = slideOutHorizontally()
    ) {
        Box(
            Modifier
                .size(150.dp)
                .background(Color.Red)
        )
    }
}
```

La API [`rememberSaveable`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/saveable/package-summary#rememberSaveable(kotlin.Array,androidx.compose.runtime.saveable.Saver,kotlin.String,kotlin.Function0)) se comporta de forma similar a `remember` porque conserva el estado durante las recomposiciones, y además lo mantiene a través de la recreación de la actividad o del proceso utilizando el mecanismo de *saved instance state*. Esto sucede, por ejemplo, cuando se rota la pantalla. `rememberSaveable` guarda automáticamente cualquier valor que pueda almacenarse en un `Bundle`. Para otros valores, se puede proporcionar un *saver* personalizado, por ejemplo un objeto parcelable, un `MapSaver` o un `ListSaver` (ver [acá](https://developer.android.com/develop/ui/compose/state#ways-to-store)).  
`rememberSaveable` recibe parámetros `input` con el mismo propósito que las `keys` usadas por `remember`. **_La cache se invalida cuando cualquiera de estos ``inputs`` cambia_**. La próxima vez que la función se recompone, `rememberSaveable` vuelve a ejecutar la _lambda_ `calculation`.

Ejemplo:

```kotlin
// rememberSaveable almacena userTypedQuery hasta que typedQuery cambie
var userTypedQuery by rememberSaveable(typedQuery, stateSaver = TextFieldValue.Saver) {
    mutableStateOf(
        TextFieldValue(text = typedQuery, selection = TextRange(typedQuery.length))
    )
}
```

### Compose ``State``
#### Primero: qué es [`snapshot`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/snapshots/Snapshot)
Es el mecanismo interno de Compose que **_registra y controla las lecturas y escrituras de estado_** para poder **_detectar cambios y disparar recomposiciones de forma eficiente y aislada_**.

#### La función [`mutableStateOf`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#mutableStateOf(kotlin.Any,androidx.compose.runtime.SnapshotMutationPolicy))

```kotlin
interface State<T : Any?>

interface MutableState<T> : State<T> {
    override var value: T
}

fun <T : Any?> mutableStateOf(
    value: T,
    policy: SnapshotMutationPolicy<T> = structuralEqualityPolicy()
): MutableState<T>
```

Crea un [`MutableState<T>`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/MutableState) observable (un tipo observable integrado con el _runtime_ de Compose; es decir, la clase `MutableState` es un **_contenedor de un único valor cuyos accesos de lectura y escritura son observados por Compose_**) inicializado con el `value` pasado como parámetro.  
**_Cualquier cambio en `value` (ya sea lectura o escritura) programa la recomposición de cualquier función composable que lea `value`._**

Ejemplo:

```kotlin
val mutableState = remember { mutableStateOf(default) }

// El delegado (con ``by``) se encarga de importar el ``getter`` y ``setter`` correspondientes.
// Esto evita tener que poner el ``.value`` de un estado cada vez que se lo necesita.
var value by remember { mutableStateOf(default) }

val (value, setValue) = remember { mutableStateOf(default) }

var username by mutableStateOf("")
  private set
```

Compose no requiere que se use `MutableState<T>` para mantener estado; también es compatible con otros tipos observables. **_Antes de leer otro tipo observable en Compose, se lo debe convertir a un `State<T>` para que las funciones composables puedan recomponerse automáticamente cuando ese estado cambie_**.  
Compose incluye funciones para crear [`State<T>`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State) a partir de tipos observables comunes usados en apps Android. Antes de usar estas integraciones, se deben agregar los [artefactos](https://developer.android.com/jetpack/androidx/releases/compose-runtime#declaring_dependencies) correspondientes.

#### La función [`produceState`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#produceState(kotlin.Any,kotlin.coroutines.SuspendFunction1))

> Ver también [produceState](#producestate) en la sección de _Side Effects_

```kotlin
interface State<T : Any?>

@Composable
fun <T : Any?> produceState(
    initialValue: T,
    producer: suspendProduceStateScope<T>.() -> Unit
): State<T>
```

**_Se usa para convertir estado externo a Compose en estado compatible con Compose_**. Por ejemplo, para traer a la composición estados basados en suscripciones externas como `Flow`, `LiveData` o `RxJava`.

Devuelve un `State` observable que **_produce valores a lo largo del tiempo sin una fuente de datos predefinida_**.  
El `producer` se lanza cuando `produceState` entra en la composición y se cancela cuando `produceState` sale de la composición. En otras palabras, inicia una _coroutine_ con *scope* en la composición que puede emitir valores hacia el `State` retornado. Aunque crea una _coroutine_, también puede usarse para **_observar fuentes de datos que no requieren suspensión_**. Para desuscribirse de esa fuente, se utiliza la función [`awaitDispose`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/ProduceStateScope#awaitDispose(kotlin.Function0)).  
El `producer` debe usar [`ProduceStateScope.value`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/ProduceStateScope#value()) para asignar nuevos valores al `State` retornado, el cual **_fusionará los valores (conflate)_**:
- No se observarán cambios si se asigna un valor igual al anterior.
- Si se asignan varios valores en sucesión rápida, **_los observadores pueden ver solo el último_**.

Ejemplo:

```kotlin
val uiState by
produceState<UiState<List<Person>>>(UiState.Loading, viewModel) {
    viewModel.people.map { UiState.Data(it) }.collect { value = it }
}

when (val state = uiState) {
    is UiState.Loading -> _root_ide_package_.org.w3c.dom.Text("Loading...")
    is UiState.Data ->
        Column {
            for (person in state.data) {
                _root_ide_package_.org.w3c.dom.Text("Hello, ${person.name}")
            }
        }
}

// --------------------------------------------------------------------------------

val uiState by produceState<TasksUiState>(
    initialValue = TasksUiState.Loading,
    key1 = lifecycle,
    key2 = tasksViewModel
) {
    lifecycle.repeatOnLifecycle(
        state = Lifecycle.State.STARTED
    ) {
        tasksViewModel.uiState.collect {
            value = it
        }
    }
}
```

#### La función [`derivedStateOf`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#derivedStateOf(kotlin.Function0))

> Consultar también [`derivedStateOf`](#derivedstateof) en la sección de _Side Effects_ y [`remember` con `key` vs `remember` junto con `derivedStateOf`](#remember-con-key-vs-remember-junto-con-derivedstateof)  en la sección de Comparativas

```kotlin
interface State<T : Any?>

fun <T : Any?> derivedStateOf(calculation: () -> T): State<T>
```

Crea un objeto `State` cuyo [`value`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State#value()) es el resultado de `calculation`.  
El resultado se cachea. Por lo tanto, leer `State.value` múltiples veces no vuelve a ejecutar `calculation`. Sin embargo, leer `State.value` hace que todos los estados consultados durante la ejecución de `calculation` se registren dentro del `Snapshot` actual. Esto garantiza que otras composables se suscriban correctamente al estado derivado cuando se lea en un contexto observado (como dentro de una función `@Composable`).

En Compose, la [recomposición](https://developer.android.com/develop/ui/compose/mental-model#recomposition) ocurre cada vez que cambia un estado observado o un parámetro de un composable. A veces, un estado o entrada puede estar cambiando más seguido de lo que la UI realmente necesita actualizar, generando recomposiciones innecesarias.

Se debe usar `derivedStateOf` cuando las entradas cambian más frecuentemente de lo que se necesita que se recomponga. Esto suele ocurrir cuando algo cambia constantemente (como la posición del _scroll_), pero el composable solo debe reaccionar cuando se cruza un cierto umbral. Para esto, `derivedStateOf` **_crea un nuevo estado observable que solo se actualizará cuando realmente sea necesario_**. En ese sentido, es similar al operador de Kotlin Flow [`distinctUntilChanged()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/distinct-until-changed.html).

Sin embargo, **_`derivedStateOf` tiene un costo_**, y se debería usar únicamente para **_evitar recomposiciones innecesarias cuando el resultado no cambió_**.

Los estados derivados sin una política de mutación personalizada se invalidan en cada cambio de sus dependencias. Para evitar invalidaciones innecesarias, se puede usar una [`SnapshotMutationPolicy`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/SnapshotMutationPolicy) adecuada mediante la sobrecarga correspondiente de `derivedStateOf`.

Ejemplo:

```kotlin
@Composable
fun CountDisplay(count: State<Int>) {
    _root_ide_package_.org.w3c.dom.Text("Count: ${count.value}")
}

@Composable
fun Example() {
    var a by remember { mutableStateOf(0) }
    var b by remember { mutableStateOf(0) }
    val sum = remember { derivedStateOf { a + b } }
    // Modificar a o b hará que CountDisplay se recomponga, pero no provocará que Example se recomponga.
    CountDisplay(sum)
}

// --------------------------------------------------------------------------------

// Cuando cambia el parámetro ``messages``, el composable MessageList se recompone. 
// ``derivedStateOf`` no afecta a esta recomposición.
@Composable
fun MessageList(messages: List<Message>) {
    Box {
        val listState = rememberLazyListState()

        LazyColumn(state = listState) {
            // ...
        }

        // Muestra el botón si el primer elemento visible está más allá del primer elemento.
        // Usamos un estado derivado almacenado en memoria para minimizar composiciones innecesarias.
        val showButton by remember {
            derivedStateOf {
                listState.firstVisibleItemIndex > 0
            }
        }

        AnimatedVisibility(visible = showButton) {
            ScrollToTopButton()
        }
    }
}
```

### *Flow*
#### La función [`collectAsState()`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#(kotlinx.coroutines.flow.StateFlow).collectAsState(kotlin.coroutines.CoroutineContext))

Es similar a `collectAsStateWithLifecycle` (ver siguiente), ya que también colecta valores de un `Flow` y los transforma en un `State` de Compose.  
Se debe usar `collectAsState` cuando se necesite escribir **_código agnóstico de la plataforma_**, en lugar de `collectAsStateWithLifecycle`, que es exclusivo de Android.

#### La función [`collectAsStateWithLifecycle()`](https://developer.android.com/reference/kotlin/androidx/lifecycle/compose/package-summary#extension-functions)

```kotlin
interface State<T : Any?>

@Composable
fun <T : Any?> StateFlow<T>.collectAsStateWithLifecycle(
    lifecycle: Lifecycle,
    minActiveState: Lifecycle.State = Lifecycle.State.STARTED,
    context: CoroutineContext = EmptyCoroutineContext
): State<T>
```

Colecta valores de un [`Flow`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/index.html) de forma consciente del ciclo de vida, permitiendo que la app conserve recursos.  
**_Representa el último valor emitido_** a través de un `State` de Compose.  
Usar esta API es la forma recomendada de colectar *flows* en aplicaciones Android.

> *Under the hood*, la implementación de `collectAsStateWithLifecycle` utiliza la API `repeatOnLifecycle`, que es la forma recomendada de recolectar *flows* en Android cuando se usa el sistema de *Views*.  
> `collectAsStateWithLifecycle` te evita escribir el *boilerplate* necesario para recolectar *flows* de manera *lifecycle-aware* desde una función composable (...)  
> La UI puede ayudar a liberar recursos recolectando el estado de UI usando `collectAsStateWithLifecycle`. El ViewModel puede hacer lo mismo produciendo el estado de UI de manera *collector-aware*. Si no hay *collectors* (por ejemplo, cuando la UI no está visible en pantalla), detener los _flows upstream_ provenientes de la capa de datos. Esto se logra usando [`.stateIn(WhileSubscribed)`](https://github.com/android/nowinandroid/blob/main/feature-author/src/main/java/com/google/samples/apps/nowinandroid/feature/author/AuthorViewModel.kt#L104) al producir el estado de UI.  
> Fuente: https://manuelvivo.dev/consuming-flows-compose || https://medium.com/androiddevelopers/consuming-flows-safely-in-jetpack-compose-cde014d0d5a3

#### La función [`asStateFlow()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/as-state-flow.html)
La extensión `.asStateFlow()` de Kotlin sirve para **_exponer una versión inmutable_** de un `MutableStateFlow`.  
Dicho de otra forma, si se hiciera `val uiState: StateFlow<LoginUiState> = _uiState`, el compilador **permitirá eso sin problemas**, porque `MutableStateFlow` **es un `StateFlow`.** Pero se sigue teniendo acceso a los métodos mutables si se hace un *cast* o se pasa la instancia a algún otro componente (`(uiState as MutableStateFlow).value = otroValor`), **_pudiendo mutar el estado desde fuera, lo cual rompe el principio de encapsulamiento_**.  
`.asStateFlow()` devuelve una instancia de `StateFlow` que **_oculta los métodos de mutabilidad_** (como `.value =`, `.emit()`, etc.), aunque internamente siga siendo la misma instancia. Con esto, se **_evita la posibilidad de que alguien haga un *cast* o acceda directamente a la mutabilidad_** desde fuera del _scope_ controlado.

Ejemplo:

```kotlin
// ViewModel
private val _uiState = MutableStateFlow(InterestsUiState(loading = true))
val uiState: StateFlow<InterestsUiState> = _uiState.asStateFlow()

private val _uiStateTwo = MutableStateFlow(ExampleUiState(loading = true))
val uiStateTwo: StateFlow<ExampleUiState> = _uiStateTwo.onStart {
    // Cargar datos iniciales o lo que sea
}.stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), ExampleUiState(loading = true))

fun onInterestChanged(interest: Int) {
    _uiState.update { uiState ->
        uiState.copy(
            interest = interest
        )
    }
}

val selectedTopics =
    interestsRepository.observeTopicsSelected().stateIn(
        viewModelScope,
        SharingStarted.WhileSubscribed(5000),
        emptySet()
    )

// --------------------------------------------------------------------------------

// Composable

val uiState by interestsViewModel.uiState.collectAsStateWithLifecycle()

val selectedTopics by interestsViewModel.selectedTopics.collectAsStateWithLifecycle()
```

### *LiveData*

#### La función [`observeAsState()`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/livedata/package-summary#(androidx.lifecycle.LiveData).observeAsState(kotlin.Any))

```kotlin
interface State<T : Any?>

@Composable
fun <R : Any?, T : R> LiveData<T>.observeAsState(initial: R): State<R>
```

Comienza a observar el [`LiveData`](https://developer.android.com/reference/kotlin/androidx/lifecycle/LiveData) y expone sus valores como un `State`. Cada vez que se postea un nuevo valor en el `LiveData`, el `State` resultante se actualizará, **_disparando recomposición_** en todos los lugares donde se lea `State.value`.

El parámetro `initial` se usa **_solo si el ``LiveData`` aún no está inicializado_**.

Si `T` es un tipo no nulo, es responsabilidad del desarrollador garantizar que el `LiveData` **_nunca contenga valores nulos_**.

Internamente, utiliza `Lifecycle` para observar los datos de forma segura. El observador se elimina automáticamente cuando el composable se desecha o cuando el [`LifecycleOwner`](https://developer.android.com/reference/kotlin/androidx/lifecycle/LifecycleOwner) llega al estado [`Lifecycle.State.DESTROYED`](https://developer.android.com/reference/kotlin/androidx/lifecycle/Lifecycle.State#DESTROYED).

Ejemplo:

```kotlin
// ViewModel

private var _textFieldHelperErrorMessage = MutableLiveData<String>()
val textFieldHelperErrorMessage: LiveData<String> = _textFieldHelperErrorMessage

// --------------------------------------------------------------------------------

// Composable

val textFieldHelperErrorMessage by viewModel.textFieldHelperErrorMessage.observeAsState(initial = "")
```

## [*Side Effects*](https://developer.android.com/develop/ui/compose/side-effects)

> **_Un efecto es una función composable que no emite UI y ejecuta side-effects cuando la composición se completa._**

Un efecto secundario es un **_cambio en el estado de la app que ocurre fuera del alcance de una función composable_**.  
Ejemplos típicos incluyen solicitudes de red, operaciones de base de datos, actualizar un ViewModel, etc.

A veces los _side-effects_ son necesarios, por ejemplo, para **_ejecutar un evento puntual_** como mostrar un _snackbar_ o navegar a otra pantalla cuando cierta condición de estado lo requiere. Estas acciones deben ejecutarse desde un entorno controlado que **_conozca el ciclo de vida_** del composable.

### `LaunchedEffect`
- Ejecuta **_funciones suspendidas_** dentro del _scope_ del propio composable. 
- Se **_dispara cuando entra en la composición y se reinicia si cambia alguno de sus keys_**. 
- Si no se proporcionan _keys_, se reiniciará en **cada recomposición**. 
- Se usa principalmente para _side-effects_ que deben estar **_vinculados al ciclo de vida del composable_** y que pueden necesitar re-ejecutarse cuando ciertos estados cambian.
- **Ejemplos comunes** :arrow_right: **_Lanzar una coroutine para cargar datos al mostrar una pantalla_**, o **_iniciar una animación que debe reiniciarse cuando cambia un valor específico_**.

### `SideEffect`
- Ejecuta un **_bloque de código después de cada recomposición exitosa_**. 
- No está **_atada a cambios de estado específicos_** dentro del composable: simplemente corre su bloque una vez que la UI se ha actualizado y es estable (ver [Recomposición & Estabilidad](#recomposición--estabilidad)).
- Se usa típicamente para _side effects_ que deben mantenerse **sincronizados con el runtime de Compose**, pero que no necesitan re-ejecutarse en función del cambio de un estado particular.
- **Ejemplo común** :arrow_right: **_Publicar estado de Compose hacia código externo que no es de Compose_**, asegurando que ese código externo siempre reciba la **_versión más reciente del estado de la UI_**.

### `DisposableEffect`
- Ejecuta **_efectos que requieren una limpieza (cleanup)_** cuando sale de la composición o cuando cambian sus *keys*.
- Se ejecuta cuando entra en composición y su bloque `onDispose { ... }` se llama automáticamente para liberar recursos asociados al efecto (por ejemplo, remover _listeners_, cancelar suscripciones, desregistrar _callbacks_, etc.).
- **Ejemplo común** :arrow_right: Integrar código que necesita **_administrar recursos manualmente_** durante el ciclo de vida del composable.

### `rememberCoroutineScope`
- Obtiene un **_scope de corrutinas consciente de la composición_** para lanzar corrutinas **_fuera del cuerpo de un composable_**, pero aún **_atadas al ciclo de vida de la composición_**.
- Las corrutinas lanzadas con este _scope_ se cancelan automáticamente cuando el composable sale de composición.
- **Ejemplo común** :arrow_right: Disparar corrutinas desde _callbacks_ de UI (por ejemplo, dentro de `onClick`), donde no se puede usar `LaunchedEffect`.

### `rememberUpdatedState`
- Permite **_capturar el valor más reciente_** dentro de un efecto **_sin provocar que dicho efecto se reinicie_** cuando ese valor cambia.
- Internamente, mantiene una referencia estable cuyo `.value` siempre refleja el último valor recibido.
- **Ejemplo común** :arrow_right: Útil cuando un `LaunchedEffect`, `DisposableEffect` u otro efecto necesita usar un valor actualizado, pero **_no se quiere que ese efecto se vuelva a ejecutar_** cada vez que ese valor cambie.

### `produceState`
> Ver también [`produceState`](#la-función-producestate)

- Convierte **_estado externo a Compose_** en **_estado observable por Compose_**.
- Inicia una **_coroutine vinculada a la composición_**, que se ejecuta mientras el composable esté en composición y se cancela al salir.
- Permite **_emitir valores hacia un `State`_** usando `value` dentro del `ProduceStateScope`.
- Es ideal para **_adaptar fuentes de datos externas_** (suscripciones, `Flow`, `LiveData`, _callbacks_, etc.) al modelo de estado de Compose sin necesidad de `collectAsState()` o equivalentes.
- **Ejemplo común** :arrow_right: **_Convertir un flujo de datos manual, un callback o una API externa en estado Compose_**, pudiendo además ejecutar lógica suspendida.

### `derivedStateOf`
> Ver también [`derivedStateOf`](#la-función-derivedstateof)

- Permite **_derivar un nuevo estado_** a partir de **_uno o varios estados existentes_**.
- Es útil cuando ciertas variables cambian **_más frecuentemente de lo que la UI realmente necesita recomponerse_**, evitando recomposiciones innecesarias.
- El resultado derivado se **_recalcula solo cuando sus dependencias cambian_**, y además **_cachea el valor_** para evitar cálculos repetidos en la misma composición.
- Funciona de forma similar a operadores como `distinctUntilChanged()` de Flow, pero a nivel de Compose.
- **Ejemplo común** :arrow_right: Derivar valores que cambian frecuentemente, como **_mostrar u ocultar un botón según la posición de scroll_**, evitando recomponer en cada pixel de movimiento.

### `snapshotFlow`
- Convierte **_cambios en el estado de Compose_** (`State`, `derivedStateOf`, etc.) en un **_Flow_**.
- Observa lecturas dentro de su _lambda_ y **emite un valor cada vez que alguno de esos estados cambie**.
- Es útil cuando se necesita **_integrar lógica basada en Flows_** con estado propio de Compose.
- Solo emite cuando el valor observado **_cambia realmente_** (**_usa comparación por igualdad_**).
- **Ejemplo común** :arrow_right: Convertir el **_estado de scroll_** o el **_valor de un campo de texto_** en un `Flow` para procesarlo con operadores de `kotlinx.coroutines.flow` (por ejemplo `debounce`, `map`, etc.).

## Algunas comparativas útiles
### ``MutableState`` vs ``StateFlow``

| ***Característica***    | `MutableState`                                                                                                                                                                                                  | `StateFlow`                                                                                                                                                                       |
|-------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Propósito               | Manejo de estado _**sincrónico**_ que dispara recomposición                                                                                                                                                     | Manejo de estado _**asincrónico**_ y observable                                                                                                                                   |
| Mutabilidad             | _**Directamente mutable**_. Se puede cambiar su valor usando su propiedad `value`                                                                                                                               | _**Solo lectura**_ (requiere `MutableStateFlow` para actualizarse)                                                                                                                |
| Asincronía              | No                                                                                                                                                                                                              | Sí                                                                                                                                                                                |
| Backing Property        | No es habitual                                                                                                                                                                                                  | Es común (usando `.asStateFlow()`)                                                                                                                                                |
| Valor inicial           | Requerido                                                                                                                                                                                                       | Requerido                                                                                                                                                                         |
| Ciclo de vida           | Ligado de forma inherente al ciclo de vida del composable. Si el composable se recompone, el `MutableState` (si fue recordado correctamente) conserva su valor                                                  | No es consciente del ciclo de vida por sí mismo. En Compose se necesita usar funciones como `collectAsStateWithLifecycle()` para un manejo adecuado del ciclo de vida             |
| Integración con Compose | Recomposición automática cuando cambia el valor                                                                                                                                                                 | Requiere `collectAsStateWithLifecycle()` (o `collectAsState()`)                                                                                                                   |
| **_Casos de uso_**      | _**Estado local de composables, valores de elementos de UI como el input de un TextField, estados de checkboxes, flags de visibilidad y otros valores que afectan directamente el renderizado del composable**_ | _**Estado del ViewModel, flujos de datos asincrónicos (ej. peticiones de red, consultas de base de datos), interacciones del usuario u otros eventos que cambian con el tiempo**_ |

<br>

### `remember` con `key` vs `remember` junto con `derivedStateOf`
Ambos tienden a hacer lo mismo: **_escuchar cambios en otro estado y luego derivar un estado a partir de él_**.

| `val something = remember(key = someKey) { … }`                                                                                                                                                                                                  | `val something = remember { derivedStateOf { … } }`                                                                                                                                                                                                                                                                                                                                                      |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `remember` es una **función *Composable*** :arrow_right: Compose analiza los parámetros que le pasamos (las `keys`) y **_en cuanto alguno de esos parámetros cambie, Compose recompondrá el composable_** donde está implementado el `remember`. | `derivedStateOf` es una **función *no-Composable*** :arrow_right: Solo actualizará el estado resultante (el de `something`) si el resultado final de `calculation` (el bloque de `derivedStateOf`) realmente cambió. Y ese **resultado final** cambia muy poco frecuentemente. Esto significa que **_Compose recompondrá cuando cambie el resultado final, no simplemente cuando cambie el parámetro_**. |

<br>

### `LaunchedEffect` vs `SideEffect`

| **Característica** | `LaunchedEffect`                                                      | `SideEffect`                                                                                                                               |
|--------------------|-----------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| **Disparador**     | Entra en la composición; se reinicia cuando cambian las *keys*        | Se ejecuta después de cada recomposición exitosa                                                                                           |
| **Recomposición**  | Se reinicia si cambian las *keys* (o en cada recomposición si no hay) | Se ejecuta después de cada recomposición; no está atado a cambios de estado específicos                                                    |
| **Caso de uso**    | *Side effects* ligados a cambios de estado específicos                | *Side effects* sincronizados con el _runtime_ de Compose                                                                                   |
| **Ejemplo**        | Obtener datos al cargar la pantalla; reiniciar una animación          | Publicar estado de Compose hacia código no-Compose; por ejemplo, hacer que un estado de Compose sea visible/accesible desde código externo |

## Animaciones en *Compose*

### Qué es *Tween*
`Tween` viene de "in-betweening", un término usado en animación tradicional para referirse a los **cuadros intermedios entre dos estados clave (_keyframes_)**.  
En Jetpack Compose, un **`tween`** es una especificación de animación que define:

- **Duración**: Cuánto tiempo tarda la animación.
- **Easing** (curva de aceleración): Cómo cambia el valor a lo largo del tiempo (lineal, aceleración, desaceleración, etc.).
- **Delay** (opcional): Espera antes de que comience la animación.

Ejemplo:

```kotlin
// Anima el valor de opacidad (alpha) de 0f a 1f o viceversa.
// Usa un tween con:
//  - Duración de 1 segundo (1000 ms);
//  - Retardo de 300 ms antes de comenzar;
//  - Y una curva de desaceleración (acelera rápido y se desacelera al final).
val animatedAlpha by animateFloatAsState(
    targetValue = if (isVisible) 1f else 0f,
    animationSpec = tween(
        durationMillis = 1000,
        easing = LinearOutSlowInEasing,
        delayMillis = 300
    )
)
```

### Animaciones *as state*
> 🔍 Ver componente en el [catálogo](https://github.com/javier-tapia/JetpackComposeCatalog/blob/master/app/src/main/java/com/cursokotlin/jetpackcomponentscatalog/animations/AnimateAsState.kt)

La función ``animateColorAsState`` recibe **un color** (``targetValue``), **una animación** que se utilizará para cambiar el valor a través del tiempo (``animationSpec``), **un _listener_ opcional** que se ejecutará cuando finalice la animación (``finishedListener``) y **un ``label`` opcional** para diferenciarla de otras animaciones en Android Studio.
Como animación se puede utilizar por ejemplo la función ``tween`` (ver [acá](#qué-es-tween)).

Cuando se cambia el ``targetValue`` proporcionado, la animación se ejecutará automáticamente. Si ya hay una animación en curso cuando cambia el color, la animación en curso se ajustará para animarse hacia el nuevo _target_.  
``animateColorAsState`` devuelve un objeto ``State``. La animación actualizará continuamente el valor de dicho objeto hasta que finalice.

Si para las animaciones de color existe la función ``animateColorAsState``, para las animaciones de tamaño está ``animateDpAsState``. Recibe los mismos parámetros, con la salvedad de que el _target_ será un tamaño en vez de un color.
A modo informativo, también existen las funciones ``animateOffsetAsState`` y ``animateFloatAsState``.

### Animaciones de visibilidad
> 🔍 Ver componente en el [catálogo](https://github.com/javier-tapia/JetpackComposeCatalog/blob/master/app/src/main/java/com/cursokotlin/jetpackcomponentscatalog/animations/AnimatedVisibility.kt)

La función ``AnimatedVisibility`` permite realizar animaciones de aparición/desaparición de un componente de forma simple y rápida.  
Entre los parámetros que recibe, tiene un ``enter`` y un ``exit``, que pueden sobreescribirse a gusto para lograr el efecto de animación deseado. Y en ``content``, irá el objeto que se quiere mostrar/ocultar.

### Animaciones de cambio de componentes (*crossfade*)
> 🔍 Ver componente en el [catálogo](https://github.com/javier-tapia/JetpackComposeCatalog/blob/master/app/src/main/java/com/cursokotlin/jetpackcomponentscatalog/animations/Crossfade.kt)

La función ``Crossfade`` permite cambiar entre dos componentes con una animación de fundido encadenado. Cada vez que cambia el estado del argumento ``targetState``, se dispara la animación, ocultando el componente "viejo" y mostrando el componente "nuevo".

### Animaciones de contenido
> 🔍 Ver componente en el [catálogo](https://github.com/javier-tapia/JetpackComposeCatalog/blob/master/app/src/main/java/com/cursokotlin/jetpackcomponentscatalog/animations/AnimatedContent.kt)

En este apartado se pueden mencionar al componente ``AnimatedContent`` y al modificador ``animateContentSize``:
- ``AnimatedContent``: Un contenedor que anima automáticamente su contenido cuando cambia ``targetState``. Su ``content`` para diferentes _target states_ se define en un mapeo entre un _target state_ y una función ``Composable``.
- ``animateContentSize``: Anima su propio tamaño cuando su modificador hijo (o el elemento ``Composable`` hijo si ya está al final de la cadena) cambia de tamaño. Esto permite que el modificador padre observe un cambio de tamaño suave, lo que resulta en un cambio visual continuo.

### *InfiniteTransition*
> 🔍 Ver componente en el [catálogo](https://github.com/javier-tapia/JetpackComposeCatalog/blob/master/app/src/main/java/com/cursokotlin/jetpackcomponentscatalog/animations/InfiniteTransition.kt)

La función ``rememberInfiniteTransition()`` permite obtener un objeto de tipo ``InfiniteTransition``, el cual se encarga de ejecutar las animaciones secundarias o hijas. Estas animaciones se pueden agregar mediante ``InfiniteTransition.animateColor``, ``InfiniteTransition.animateFloat`` o ``InfiniteTransition.animateValue``. Las animaciones secundarias comenzarán a ejecutarse en cuanto entren en la composición y no se detendrán hasta que se eliminen de ella.

## Previews

### Live Template :arrow_right: `prevCol`

```kotlin
// Description: Creates a CollectionPreviewParameterProvider

class $NAME$: PreviewParameterProvider < $TYPE$> {
    override val values = sequenceOf(
        $END$
    )
}
```

### Usando `PreviewParameterProvider`

```kotlin
// ExampleParameterProvider.kt

data class ExampleParameters(
    val name: String?,
    val lastName: String?,
    val onTextFieldValueChange: (String) -> Unit
)

class ExamplePreviewParameterProvider : PreviewParameterProvider<ExampleParameters> {
    override val values = sequenceOf(
        ExampleParameters(
            name = "Javi",
            lastName = "Fulanito",
            onTextFieldValueChange = { }
        ),
        ExampleParameters(
            name = "Juan",
            lastName = "Polainas",
            onTextFieldValueChange = { }
        ),
        ExampleParameters(
            name = "John",
            lastName = "Connor",
            onTextFieldValueChange = { }
        )
    )
}

// --------------------------------------------------------------------------------

// Example.kt

@Composable
fun Example(
    name: String?,
    lastName: String?,
    val onTextFieldValueChange: (String) -> Unit
) {
    // Do something
}

@Preview
@Composable
private fun ExamplePreview(
    @PreviewParameter(ExamplePreviewParameterProvider::class)
    exampleParameters: ExampleParameters
) {
    Example(
        name = exampleParameters.name,
        lastName = exampleParameters.lastName,
        onTextFieldValueChange = exampleParameters.onTextFieldValueChange
    )
}
```
