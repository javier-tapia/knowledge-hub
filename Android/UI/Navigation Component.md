<h1><i>Navigation Component</i></h1>

> :warning: El componente de Navigation que se describe para Compose, ya quedó deprecado. Ya está productiva la versión de Navigation3.
>
> 🔍 Revisar lo relativo a la Navegación en Compose en el [Lab de Navigation](https://github.com/javier-tapia/android-and-kotlin-lab-chronicles/tree/master/app/src/main/java/com/example/android_and_kotlin_lab_chronicles/experiments/jetpack_compose/navigation)

***Index***:
<!-- TOC -->
  * [✅ ¿Qué se usa en Jetpack Compose?](#-qué-se-usa-en-jetpack-compose)
  * [🚧 ¿Y si se está trabajando un proyecto híbrido (XML + Compose)?](#-y-si-se-está-trabajando-un-proyecto-híbrido-xml--compose)
  * [*Navigation* con XML's](#navigation-con-xmls)
  * [Navegar entre destinos](#navegar-entre-destinos)
  * [Pasar datos entre destinos](#pasar-datos-entre-destinos)
  * [``NavigationUI``](#navigationui)
  * [*Nested graphs*](#nested-graphs)
  * [Acciones globales](#acciones-globales)
<!-- TOC -->

---

## ✅ ¿Qué se usa en Jetpack Compose?
1. Google lanzó una versión del componente de _Navigation_ pensada específicamente para Compose:  

```kotlin
implementation("androidx.navigation:navigation-compose:<version>")
```

2. Se usa ``rememberNavController()`` para crear un ``NavHostController``, que será el encargado de **_gestionar la navegación_** entre los ``Composables``.

3. Se define un ``NavHost``, dentro del árbol de composables, que **_provee el lugar en el cual se realizará la navegación_**. Como argumentos puede recibir, entre otros, un ``NavHostController`` y un ``startDestination``.

4. En Compose, **_se navega con rutas_**, las cuales son pasadas al método ``navigate`` del ``NavHostController``.  
A su vez, con el mismo método ``navigate`` se pueden **_pasar datos primitivos a otra pantalla_**. Luego, en la pantalla destino, se puede utilizar un objeto ``NavBackStackEntry`` y llamar a su método ``toRoute``, lo cual permite acceder a los datos que se pasaron por parámetros desde la pantalla inicial.

Por otro lado, si bien Google no lo recomienda, también es posible pasar datos _custom_ o complejos (por ejemplo, una _data class_). Para eso, se puede agregar el _plugin_ de ``org.jetbrains.kotlin.plugin.parcelize`` tanto en el _build.gradle_ del proyecto como de la _app_; y luego, se debe agregar la etiqueta ``@Parcelize`` a la _data class_ correspondiente, además de la etiqueta ``@Serializable``. A su vez, la pantalla destino deberá proveer un objeto de tipo ``NavType`` que servirá para mapear el objeto _custom_ al tipo de datos que requiere el sistema para propagarlo como argumento.

**Ejemplo básico**:

```kotlin
val navController = rememberNavController()
NavHost(
    navController = navController,
    startDestination = Login
) {
    composable<Login> {
        LoginScreen(
            navigateToDetail = { navController.navigate(Home) },
            snackbarHostState = snackbarHostState,
            coroutineScope = coroutineScope,
        )
    }

    composable<Home> {
        HomeScreen(
            navigateBack = { navController.popBackStack() },
            navigateToDetail = { navController.navigate(Detail(id = it)) }
        )
    }
    
    composable<Detail> { navBackStackEntry ->
        val detail = navBackStackEntry.toRoute<Detail>()
        DetailScreen(
            id = detail.id,
            navigateToSettings = { navController.navigate(Settings(it)) }
        )
    }
}
```

## 🚧 ¿Y si se está trabajando un proyecto híbrido (XML + Compose)?
- Se puede seguir usando ``NavHostFragment`` y ``nav_graph.xml``.
- Se puede integrar Compose dentro de los destinos usando ``ComposeView``.

**_En resumen:_** 

| Antes (View/XML)              | Ahora (Jetpack Compose)   |
|-------------------------------|---------------------------|
| `NavHostFragment` en XML      | `NavHost` como composable |
| `findNavController()`         | `rememberNavController()` |
| `nav_graph.xml` con flechitas | Todo en código Kotlin     |
| `navigate(R.id.action_...)`   | `navigate(route = route)` |

---

## *Navigation* con XML's
El **componente** ***Navigation*** consta de tres partes clave que se describen a continuación:  
- **Gráfico de navegación** (***Navigation graph***): Es un recurso XML que contiene **toda la información relacionada con la navegación** en una ubicación centralizada. Esto incluye todas las áreas de contenido individuales dentro de la *app*, llamadas **destinos** (***destinations***), así como las posibles rutas que un usuario puede tomar a través de la *app*, llamadas **acciones** (***actions***).  
- **``NavHost``**: Es un **contenedor vacío que muestra los destinos del gráfico de navegación**. El componente *Navigation* contiene una implementación ``NavHost`` predeterminada, **``NavHostFragment``**, que muestra destinos de *fragments*.  
- **``NavController``**: Es un **objeto que administra la navegación de la** ***app*** dentro de un ``NavHost``. Orquesta el intercambio de contenido de destino en el objeto ``NavHost`` a medida que los usuarios se mueven a través de la *app*.

A su vez, como principios de navegación generales (no se aplican solo al usar el componente *Navigation*), cabe destacar lo siguiente: toda *app* debe tener un **destino de inicio fijo**; los botones de **Atrás** (***Back***) y **Arriba** (***Up***), funcionan de la misma manera dentro de la *app*, con la diferencia de que **el botón ***Up*** nunca sale de la aplicación**; en caso de utilizar **vínculos directos** (***deep linking***) para navegar a una pantalla de la *app*, se debe crear una **pila** (***back stack***) **artificial**, como si se hubiera llegado a esa pantalla de forma normal.  
  Para usar el componente *Navigation* con Kotlin, se agregan las dependencias en el *build.gradle*:

```kotlin
implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
implementation "androidx.navigation:navigation-ui-ktx:$nav_version"
```

Para agregar un nuevo gráfico de navegación, se crea un nuevo ***resource file*** de tipo *Navigation*. Para agregar un *NavHostFragment*, se agrega en el *xml* de esta forma:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.appcompat.widget.Toolbar
        .../>
    
    <!-- Se agrega un NavHostFragment -->
    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"

        app:defaultNavHost="true"
        app:navGraph="@navigation/nav_graph" />

    <com.google.android.material.bottomnavigation.BottomNavigationView
        .../>

</androidx.constraintlayout.widget.ConstraintLayout>
```

Para establecer un destino como el **destino de inicio**, se hace clic derecho sobre el destino y luego ***Set as Start Destination***. En el editor de navegación, **las acciones se representan con flechas**, aunque para navegar realmente entre los destinos **hace falta escribir el código** correspondiente. Estas acciones, en la vista *xml* del gráfico, se representan con elementos ***``<action>``***. Como mínimo, una acción contiene su propio ID y el ID del destino (*fragment* o *activity*) al que se debería dirigir a un usuario.

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:android="http://schemas.android.com/apk/res/android"
    app:startDestination="@id/blankFragment">
    <fragment
        android:id="@+id/blankFragment"
        android:name="com.example.cashdog.cashdog.BlankFragment"
        android:label="fragment_blank"
        tools:layout="@layout/fragment_blank" >
        <action
            android:id="@+id/action_blankFragment_to_blankFragment2"
            app:destination="@id/blankFragment2" />
    </fragment>
    <fragment
        android:id="@+id/blankFragment2"
        android:name="com.example.cashdog.cashdog.BlankFragment2"
        android:label="fragment_blank_fragment2"
        tools:layout="@layout/fragment_blank_fragment2" />
</navigation>
```

Para recuperar un ``NavController`` (el objeto que administra la navegación de una app dentro de un elemento ``NavHost``), se puede usar uno de los siguientes métodos (en Kotlin):
- ***``Fragment.findNavController()``***
- ***``View.findNavController()``***
- ***``Activity.findNavController(viewId: Int)``***

```kotlin
class MainActivity : AppCompatActivity() {

  private lateinit var navController: NavController

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    // Otras configuraciones

    navController = findNavController(R.id.nav_host_fragment)
  }
}
```

## Navegar entre destinos
Para navegar entre destinos, Android recomienda utilizar el *plugin* de *Gradle* llamado ***Safe Args***, que permite navegar con **seguridad de tipo** y **pasar argumentos** entre los destinos. Para agregar *Safe Args* al proyecto, se incluye la siguiente *classpath* en el archivo *build.gradle* de nivel superior (a nivel de proyecto):

```kotlin
buildscript {
  repositories {
    google()
  }
  dependencies {
    def nav_version = "2.3.0"
    classpath "androidx.navigation:navigation-safe-args-gradle-plugin:$nav_version"
  }
}
```

También se agrega el complemento correspondiente:

```kotlin
apply plugin: "androidx.navigation.safeargs.kotlin"
```

Después de habilitar *Safe Args*, el complemento genera código que contiene clases y métodos para cada acción definida. En cada acción, *Safe Args* también genera una clase **para cada destino de origen**, que es el destino desde el que se origina la acción. El nombre de clase generada es una combinación del nombre de la clase del destino de origen y la palabra ***Directions***. Por ejemplo, si el destino se llama ***SpecifyAmountFragment***, la clase generada se llama ***SpecifyAmountFragmentDirections***. La clase generada contiene **un método estático para cada acción definida** en el destino de origen. Este método toma cualquier parámetro de acción definido como argumento y retorna un objeto ***NavDirections*** que se puede pasar al **método** ***``navigate()``***.

```kotlin
override fun onClick(view: View) {
  val action =
    SpecifyAmountFragmentDirections
      .actionSpecifyAmountFragmentToConfirmationFragment()
  view.findNavController().navigate(action)
}
```

Si bien *Safe Args* es lo recomendado, también se puede navegar entre destinos **a través de ID’s** o con ***DeepLinkRequest***. En el primer caso, ***``navigate(int)``*** toma el ID ya sea de una acción o directamente de un destino (se recomienda usar acciones siempre que sea posible, ya que se puede remplazar por operaciones con *SafeArgs*, brindan información adicional y también permiten animar las transiciones).

```kotlin
viewTransactionsButton.setOnClickListener { view ->
  view.findNavController().navigate(R.id.viewTransactionsAction)
}
```

En el segundo caso, se puede usar **``navigate(NavDeepLinkRequest)``** para navegar directamente a un destino de vínculo directo implícito.

```kotlin
val request = NavDeepLinkRequest.Builder
  .fromUri("android-app://androidx.navigation.app/profile".toUri())
  .build()
findNavController().navigate(request)
```

Además de Uri, ``NavDeepLinkRequest`` también admite vínculos directos con acciones y tipos de MIME. Para agregar una acción a la solicitud, se usa ***``fromAction()``*** o ***``setAction()``***. Para agregar un tipo de MIME a una solicitud, se usa ***``fromMimeType()``*** o ***``setMimeType()``***.

## Pasar datos entre destinos
*Navigation* permite adjuntar datos a una operación de navegación si se definen los argumentos de un destino. Por ejemplo, el destino de un perfil de usuario podría tomar como argumento el ID de un usuario para determinar a quién mostrar el contenido. En general, se debe optar por pasar **sólo la cantidad mínima de datos entre destinos**. Por ejemplo, pasar una clave a fin de recuperar un objeto en lugar de pasar el objeto en sí, ya que el espacio total para los estados guardados en Android es limitado.  
Si se necesita pasar **grandes cantidades de datos**, es preferible utilizar un ***ViewModel*** **compartido**: dos *fragments* pueden compartir un *ViewModel* usando su ámbito de actividad (*activity scope*) para manejar la comunicación.

```kotlin
class SharedViewModel : ViewModel() {
  val selected = MutableLiveData<Item>()

  fun select(item: Item) {
    selected.value = item
  }
}

class MasterFragment : Fragment() {

  private lateinit var itemSelector: Selector

  // Use the 'by activityViewModels()' Kotlin property delegate
  // from the fragment-ktx artifact
  private val model: SharedViewModel by activityViewModels()

  override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    itemSelector.setOnClickListener { item ->
      // Update the UI
    }
  }
}

class DetailFragment : Fragment() {

  // Use the 'by activityViewModels()' Kotlin property delegate
  // from the fragment-ktx artifact
  private val model: SharedViewModel by activityViewModels()

  override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    model.selected.observe(viewLifecycleOwner, Observer<Item> { item ->
      // Update the UI
    })
  }
}
```

Para pasar datos entre destinos, se define el argumento agregándolo al destino que lo recibe (desde el panel ***Attributes***).

```xml
<fragment android:id="@+id/myFragment" >
   <argument
       android:name="myArg"
       app:argType="integer"
       android:defaultValue="0" />
</fragment>
```

Todas las acciones que navegan al destino usan valores predeterminados y argumentos a nivel de destino. Si es necesario, **se puede sobreescribir el valor predeterminado de un argumento** (o definir uno si todavía no existe). **Para eso, se define un argumento a nivel de la acción**. Este argumento debe tener el mismo nombre y tipo que el declarado en el destino. El código XML que se muestra a continuación declara una acción con un argumento que sobreescribe el argumento a nivel de destino del ejemplo anterior:

```xml
<action android:id="@+id/startMyFragment"
    app:destination="@+id/myFragment">
    <argument
        android:name="myArg"
        app:argType="integer"
        android:defaultValue="1" />
</action>
```

Como ya se dijo, lo recomendado para navegar y pasar datos entre destinos es ***Safe Args***, ya que garantiza **seguridad de tipo** (***type-safety***). Así como crea una clase por cada destino donde se origina una acción (el nombre es el destino de origen más la palabra *Directions*), también se crea una clase para el destino de recepción. El nombre de esta clase es el nombre del destino, unido a la palabra ***Args***. Por ejemplo, si el *fragment* de destino se llama ***ConfirmationFragment***, la clase generada se llamará ***ConfirmationFragmentArgs***. Y también se crea una clase interna para cada acción que se usa a fin de pasar el argumento, cuyo nombre se basa en la acción. Por ejemplo, si la acción se llama ***confirmationAction***, la clase se llamará ***ConfirmationAction***. Si la acción contiene argumentos sin un ***defaultValue***, se debe usar la clase de acción asociada para configurar el valor de los argumentos.  
En el siguiente ejemplo, se muestra cómo utilizar estos métodos para configurar un argumento y pasarlo al **método** ***``navigate()``***:

```kotlin
override fun onClick(v: View) {
  val amountTv: EditText = view!!.findViewById(R.id.editTextAmount)
  val amount = amountTv.text.toString().toInt()
  val action = SpecifyAmountFragmentDirections.confirmationAction(amount)
  v.findNavController().navigate(action)
}
```

En el código del destino de recepción, se usa el **método** ***``getArguments()``*** a fin de recuperar el *bundle* y usar su contenido. Cuando se usan las **dependencias** ***-ktx***, con Kotlin también se puede usar el **delegado de propiedades** ***``by navArgs()``*** para acceder a los argumentos:

```kotlin
val args: ConfirmationFragmentArgs by navArgs()

override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
  val tv: TextView = view.findViewById(R.id.textViewAmount)
  val amount = args.amount
  tv.text = amount.toString()
}
```

En algunos casos, por ejemplo, **si no se usa** ***Gradle***, no se puede usar el complemento *Safe Args*. En esas situaciones, se pueden utilizar objetos ***Bundle*** para pasar datos de forma directa. Se crea un objeto *Bundle* y se lo pasa al destino usando ***``navigate()``***.

```kotlin
val bundle = bundleOf("amount" to amount)
view.findNavController().navigate(R.id.confirmationAction, bundle)
```

Y en el código del destino de recepción, se usa el **método** ***``getArguments()``*** para recuperar el *Bundle*:

```kotlin
val tv = view.findViewById<TextView>(R.id.textViewAmount)
tv.text = arguments?.getString("amount")
```

## ``NavigationUI``
El componente *Navigation* incluye una clase ``NavigationUI``. Esta clase contiene métodos estáticos que administran la navegación con la barra superior de la *app* (***top bar***), el panel lateral de navegación (***navigation drawer***) y la navegación inferior (***bottom navigation***). ``NavigationUI`` también proporciona asistentes para **vincular destinos a componentes de UI controlados por el menú**: contiene un método asistente, ***``onNavDestinationSelected()``***, que toma un elemento ``MenuItem`` junto con el elemento ``NavController`` que aloja el destino asociado. Si el elemento ***``id``*** de MenuItem **coincide** con el elemento ***``id``*** del destino, ``NavController`` puede entonces navegar a ese destino.  
- ***Top bar***: ``NavigationUI`` contiene métodos que actualizan automáticamente el contenido de la barra superior a medida que los usuarios navegan por la *app*. Por ejemplo, **se pueden usar las etiquetas (***labels***) del destino del gráfico de navegación para mantener actualizado el título de la barra superior** de la *app*, usando el **método** ***``setupWithNavController``***. Cuando se usa ``NavigationUI`` con alguna de las implementaciones de la barra superior (***Toolbar***, ***CollapsingToolbarLayout*** o ***ActionBar***), la etiqueta que se adjunta a los destinos puede ser automáticamente populada a partir de los argumentos proporcionados al destino, usando la sintaxis ***``{argName}``*** en la etiqueta. ``NavigationUI`` usa un objeto ***AppBarConfiguration*** para **administrar el comportamiento del botón** ***Navigation*** en la esquina superior izquierda del área de visualización de la *app*. El comportamiento del botón *Navigation* cambia si el usuario se encuentra en un **destino de nivel superior** (la raíz, o el destino de nivel más alto, en un conjunto de destinos relacionados de manera jerárquica). Los destinos de nivel superior no muestran un botón ***Arriba*** en la barra superior de la *app*, ya que no existe un destino superior a este. De forma predeterminada, el destino de inicio de la *app* es el único destino de nivel superior.  
- ***Navigation drawer***: El ícono del panel lateral (***drawer icon***) se muestra en todos los destinos de nivel superior que usan un ``DrawerLayout``. Para agregar un panel lateral de navegación, primero se declara un ``DrawerLayout`` como vista raíz. Dentro del ``DrawerLayout``, se agrega un diseño para el contenido principal de la UI y otra vista que tenga el contenido del panel lateral de navegación (``NavigationView``). Luego, se conecta el ``DrawerLayout`` al gráfico de navegación pasándoselo a ***AppBarConfiguration***. Y por último, en el método *``onCreate()``* de la *activity* se llama a ***``setupWithNavController()``***.  
- ***Bottom Navigation***: ``NavigationUI`` también puede controlar la navegación inferior. Cuando un usuario selecciona un elemento de menú, ``NavController`` llama a ***``onNavDestinationSelected()``*** y actualiza automáticamente el elemento seleccionado en la barra de navegación inferior. Para crear una barra de navegación inferior, primero se define en el XML de la *activity* principal; luego, se llama a ***``setupWithNavController()``*** desde el método *``onCreate()``* de la *activity* principal.

La interacción con ``NavController`` es el método principal para navegar entre destinos. El ``NavController`` es responsable de reemplazar el contenido del *NavHost* con el destino nuevo. En muchos casos, los elementos de la UI (por ejemplo, una barra superior u otros controles de navegación persistentes como *BottomNavigationBar*) permanecen fuera del *NavHost* y se deben actualizar a medida que se navega entre destinos. ``NavController`` ofrece la **interface** ``OnDestinationChangedListener`` que se llama cuando **el destino actual o los argumentos de **``NavController``** cambian**. Es posible registrar un nuevo objeto de escucha mediante el **método** ***``addOnDestinationChangedListener()``***.  
``NavigationUI`` usa ``OnDestinationChangedListener`` para hacer que estos componentes comunes de la UI reconozcan la navegación. Sin embargo, hay que tener en cuenta que **también se puede usar el elemento ``OnDestinationChangedListener`` por sí solo, con el fin de que cualquier UI personalizada o lógica de negocios esté al tanto de los eventos de navegación**. Por ejemplo, tal vez se tengan elementos de UI comunes que se quieran mostrar en algunas áreas de la *app* y ocultar en otras. Con un ``OnDestinationChangedListener`` propio, se pueden ocultar o mostrar selectivamente estos elementos de la UI en función del destino objetivo, como se muestra en el siguiente ejemplo:

```kotlin
navController.addOnDestinationChangedListener { _, destination, _ ->
  if (destination.id == R.id.full_screen_destination) {
    toolbar.visibility = View.GONE
    bottomNavigationView.visibility = View.GONE
  } else {
    toolbar.visibility = View.VISIBLE
    bottomNavigationView.visibility = View.VISIBLE
  }
}
```

## *Nested graphs*
Por lo general, los flujos de acceso, los asistentes (*wizards*) y otros subflujos de la *app* se representan mejor como **gráficos de navegación anidados** (***nested graphs***). Si se anidan flujos de subnavegación autónomos de esta manera, el flujo principal de la UI de la *app* será más fácil de comprender y administrar. Además, los gráficos anidados son reutilizables. También proporcionan un nivel de encapsulación, es decir, que los destinos fuera del gráfico anidado no tienen acceso directo a ninguno de los destinos dentro del gráfico anidado. En su lugar, deberían tener un elemento *``navigate()``* que los dirija al propio gráfico anidado, donde la lógica interna puede cambiar sin afectar el resto del gráfico. Esto es útil, por ejemplo, para verificar si hay un usuario registrado. Si el usuario no está registrado, se lo puede dirigir a la pantalla de registro dentro del gráfico anidado. Esto es lo que se conoce como **navegación condicional**.  
Para agrupar destinos en un gráfico anidado, se mantiene presionada la **tecla** ***Shift*** y se hace clic en los destinos que se desean incluir en el gráfico anidado. Luego, se hace clic derecho para abrir el menú contextual y se selecciona ***Move to Nested Graph > New Graph***.  
Además, dentro de un gráfico de navegación se puede hacer referencia a otros gráficos mediante ***``include``***. Si bien esto es funcionalmente lo mismo que usar un gráfico anidado, *``include``* permite usar gráficos de otros módulos del proyecto o de proyectos de librería.

## Acciones globales
Se puede usar una **acción general o global** (***global action***) para crear una acción común que varios destinos puedan utilizar. Por ejemplo, quizás se desee agregar botones en distintos destinos para navegar a la misma pantalla principal de la *app*. En el Editor de *Navigation*, una acción general se representa mediante **una flecha pequeña que apunta al destino asociado**. A fin de usar una acción general en el código, se pasa el ID de recurso de la acción general al **método** ***``navigate()``*** para cada elemento de la UI, como se muestra en el siguiente ejemplo:

```kotlin
viewTransactionButton.setOnClickListener { view ->
  view.findNavController().navigate(R.id.action_global_mainFragment)
}
```
