<h1><i>Networking & Monitoring</i></h1>

***Index***:
<!-- TOC -->
  * [*Retrofit*](#retrofit)
    * [🚀 Cheatsheet Retrofit](#-cheatsheet-retrofit)
    * [Dependencias y permisos](#dependencias-y-permisos)
    * [Modelo de Respuesta](#modelo-de-respuesta)
    * [*Interface API service*](#interface-api-service)
    * [Cliente OkHttp](#cliente-okhttp)
    * [Interceptores de OkHttp: diferencias y usos recomendados](#interceptores-de-okhttp-diferencias-y-usos-recomendados)
      * [🛂 *Application Interceptor*](#-application-interceptor)
      * [🌐 *Network Interceptor*](#-network-interceptor)
    * [Instancia de Retrofit](#instancia-de-retrofit)
    * [Manejo de respuestas y errores](#manejo-de-respuestas-y-errores)
  * [*Ktor* (cliente)](#ktor-cliente)
    * [🚀 Cheatsheet Ktor Client](#-cheatsheet-ktor-client)
    * [Dependencias y permisos](#dependencias-y-permisos-1)
    * [Modelo de respuesta con KotlinX Serialization](#modelo-de-respuesta-con-kotlinx-serialization)
    * [Configuración del cliente HTTP](#configuración-del-cliente-http)
    * [Realizar solicitudes](#realizar-solicitudes)
    * [Manejo de respuestas y errores](#manejo-de-respuestas-y-errores-1)
  * [*Sentry*](#sentry)
    * [Características Principales](#características-principales)
    * [Cómo se integra con Android](#cómo-se-integra-con-android)
      * [1. Configuración en la Plataforma Sentry](#1-configuración-en-la-plataforma-sentry)
      * [2. Adición de Dependencias Gradle](#2-adición-de-dependencias-gradle)
      * [3. Inicialización del SDK](#3-inicialización-del-sdk)
      * [4. Integración con *Gradle Plugin*](#4-integración-con-gradle-plugin)
      * [5. Monitoreo NDK (Opcional)](#5-monitoreo-ndk-opcional)
      * [6. Uso del Asistente (*Sentry Wizard*)](#6-uso-del-asistente-sentry-wizard)
  * [*Segment*](#segment)
    * [Características Principales](#características-principales-1)
    * [Cómo se integra con Android](#cómo-se-integra-con-android-1)
      * [1. Configuración del Origen (*Source*) en Segment](#1-configuración-del-origen-source-en-segment)
      * [2. Integración del SDK en Gradle](#2-integración-del-sdk-en-gradle)
      * [3. Inicialización en la App](#3-inicialización-en-la-app)
      * [4. Implementación de Seguimiento (*Tracking*)](#4-implementación-de-seguimiento-tracking)
      * [5. Activación de Destinos (*Destinations*)](#5-activación-de-destinos-destinations)
<!-- TOC -->

---

## *Retrofit*
> 🔍 Referencias:  
> https://square.github.io/retrofit/  
> https://github.com/square/retrofit  
> https://square.github.io/okhttp/  
> https://johncodeos.com/how-to-make-post-get-put-and-delete-requests-with-retrofit-using-kotlin/

Es una librería con **seguridad de tipo** (_type-safe_) para **_realizar solicitudes HTTP_** y **_mapear las respuestas_** a objetos previamente modelados (con _data class_ en Kotlin).  
No tiene injerencia sobre _cache_, _retries_ ni _logging_. Estas responsabilidades recaen completamente en [OkHttp](#cliente-okhttp), no en Retrofit.

### 🚀 Cheatsheet Retrofit
1. **Definir el modelo de datos**

```kotlin
data class UserDto(
    val id: String,
    val name: String
)
```

2. **Definir interfaz del servicio**

```kotlin
interface UserApi {
    @GET("users/{id}")
    suspend fun fetchUser(
        @Path("id") id: String
    ): Response<UserDto>
}
```

3. **Crear instancia de Retrofit**

```kotlin
val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .addConverterFactory(GsonConverterFactory.create())
    .build()
```

4. **Crear implementación del servicio (una por cada interfaz en caso de haber más)**

```kotlin
val api = retrofit.create(UserApi::class.java)
```

5. **Ejecutar _request_ + Manejo de respuesta y errores**

```kotlin
suspend fun getUser(id: String): Result<UserDto> {
    return try {
        // Solicitud a la red
        val response = api.fetchUser(id)

        // Otras operaciones
    } catch (e: Exception) {
        // Gestionar errores
    }
}
```

### Dependencias y permisos
Agregar las dependencias necesarias en el archivo ``libs.versions.toml``:

```toml
[versions]
kotlinSerialization = "{VERSION}"
retrofit = "{VERSION}"
okhttp = "{VERSION}"

[libraries]
kotlinx-serialization-json = { module = "org.jetbrains.kotlinx:kotlinx-serialization-json", version.ref = "kotlinSerialization" }
retrofit = { module = "com.squareup.retrofit2:retrofit", version.ref = "retrofit" }
retrofit-converter-kotlinx = { module = "com.squareup.retrofit2:converter-kotlinx-serialization", version.ref = "retrofit" }
okhttp-logging = { module = "com.squareup.okhttp3:logging-interceptor", version.ref = "okhttp" }
```

Implementar las dependencias en el archivo ``build.gradle.kts`` del módulo que corresponda:

```kotlin
// Kotlinx Serialization
implementation(libs.kotlinx.serialization.json)

// Retrofit & OkHttp
implementation(libs.retrofit)
implementation(libs.retrofit.converter.kotlinx)
implementation(libs.okhttp.logging)
```

Agregar el permiso de internet en el ``Manifest``:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

### Modelo de Respuesta
> :warning: Es importante destacar que Retrofit no trae soporte nativo para KotlinX Serialization, ya que no es parte de Retrofit, sino un _converter_ externo. Gson, en cambio, sí tiene soporte oficial porque es el _converter_ por defecto recomendado históricamente.  
> Para usar el converter de KotlinX Serialization, se requiere agregar el módulo ``converter-kotlinx-serialization``.

A las propiedades de las _data class_ que modelan la respuesta del servicio, se les puede agregar una _annotation_ (por ejemplo, ``@SerializedName`` para Gson o ``@SerialName`` para KotlinX Serialization) y pasarle el nombre del atributo. Esto permite que el modelo sea agnóstico a los nombres "reales" y se pueda reutilizar más fácilmente. Dichas anotaciones son opcionales si el nombre de la propiedad coincide con el JSON.  
Además, en caso de usar el ``Converter`` propio de KotlinX Serialization, se debe anotar la clase con ``@Serializable``.

```kotlin
@Serializable
data class UserDto(
    @SerialName("user_id")
    val userId: String,
    @SerialName("name")
    val name: String,
    @SerialName("nickname")
    val nickname: String,
    @SerialName("followers")
    val followers: Int,
    @SerialName("following")
    val following: List<String>,
    @SerialName("user_type")
    val userType: Int,
)
```

### *Interface API service*
Crear una interfaz que declare los métodos para realizar las solicitudes HTTP y el tipo de retorno, el cual puede estar encapsulado en un ``Response<T>`` (ver [Manejo de respuestas y errores](#manejo-de-respuestas-y-errores)). Esto es opcional y se podría usar un tipo definido por el desarrollador directamente, pero ``Response`` sirve para leer _headers_, verificar si la respuesta fue exitosa con ``isSuccessful``, acceder al código de respuesta con ``code()`` y manejar errores de forma más controlada.  
Se anotan con el verbo de la llamada y, opcionalmente, se pueden pasar parámetros como _headers_, _query params_, _body_ (para los ``POST``, ``PUT`` o ``PATCH``), entre otros.

```kotlin
interface SampleApiService {
    @POST("update_user")
    suspend fun updateUser(
        @Header("Journey-Id") journeyId: String,
        @Query("session_id") sessionId: String?,
        @Body sampleBody: SampleBody?,
    ): Response<SampleUpdateResponse>

    @GET("users/{id}")
    suspend fun fetchUser(
        @Header("Journey-Id") journeyId: String,
        @Query("session_id") sessionId: String?,
        @Path("id") id: String,
    ): Response<SampleFetchResponse>
}
```

### Cliente OkHttp
> ⚠️ Importante: Para poder utilizar ``BuildConfig``, se debe agregar la _flag_ ``buildConfig = true`` dentro del bloque ``android.buildFeatures`` en el archivo ``build.gradle.kts(:app)``

Retrofit usa OkHttp internamente como cliente HTTP, NO lo reemplaza. Por eso es posible personalizarlo antes de pasárselo a Retrofit.  
Permite configurar el comportamiento real de las conexiones HTTP, incluyendo interceptores, _timeouts_, _logging_, políticas de reintento, _cache_ y _headers_ globales (ver apartado siguiente sobre [interceptores](#interceptores-de-okhttp-diferencias-y-usos-recomendados)).

Ejemplo:

```kotlin
val okHttpClient = OkHttpClient.Builder()
    .connectTimeout(20, TimeUnit.SECONDS)
    .readTimeout(20, TimeUnit.SECONDS)
    .writeTimeout(20, TimeUnit.SECONDS)
    .addInterceptor(HttpLoggingInterceptor().apply {
        level = if (BuildConfig.DEBUG) {
            HttpLoggingInterceptor.Level.BODY
        } else {
            HttpLoggingInterceptor.Level.NONE
        }
    })
    .build()
```

### Interceptores de OkHttp: diferencias y usos recomendados
OkHttp permite agregar dos tipos de interceptores, que se ejecutan en distintos momentos del ciclo de una _request_.

| Característica                | 🛂 **Application Interceptor**                                                                            | 🌐 **Network Interceptor**                                                                                       |
|-------------------------------|-----------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------|
| **Cuándo se ejecuta**         | Antes de cualquier acceso a red                                                                           | Justo antes y después de tocar la red (socket)                                                                   |
| **Redirecciones**             | ❌ Se ejecuta **una sola vez**                                                                             | ✔️ Se ejecuta **por cada redirección**                                                                           |
| **Cache**                     | ✔️ Puede ver respuestas de cache                                                                          | ❌ **No** se ejecuta **cuando OkHttp responde desde el cache** (**sin tocar la red**)                             |
| **Logging**                   | Logging "lógico" (lo que el código envía)                                                                 | Logging "real" (lo que realmente se envió/recibió en red)                                                        |
| **Modificación de request**   | ✔️ Ideal para agregar headers o reescribir requests                                                       | ✔️ Puede modificar request pero ya "casi finalizada"                                                             |
| **Modificación de respuesta** | ✔️ Puede modificarla (incluyendo respuestas cacheadas), pero no es lo habitual                            | ✔️ Puede modificarla (solo cuando viene de red)                                                                  |
| **Casos ideales**             | - Headers globales<br>- Auth tokens<br>- Retries lógicos<br>- Logging de negocio<br>- Reescritura general | - TLS / Certificado<br>- Logging de red real<br>- Inspección de proxies/servidor<br>- Manejo específico de cache |
| **Acceso al socket**          | ❌ No                                                                                                      | ✔️ Sí                                                                                                            |
| **Uso más común**             | Interceptor general de la app                                                                             | Interceptor para debugging, inspección y validaciones profundas de red                                           |
| **Ejecutado sobre**           | La **request original**                                                                                   | La **request final** (después de compresión, headers automáticos, etc.)                                          |

#### 🛂 *Application Interceptor*
Se ejecuta **_antes de que la request llegue a la red_**, actuando en la capa más externa del OkHttpClient. En resumen: es el **_interceptor a usar para lógica de la aplicación_**, sin preocuparte por detalles de transporte o red.

Es ideal para:
- Agregar **_headers_ globales** 
- Manejar **autenticación** (_Tokens_, API _Keys_, _Bearer_, etc.)
- **_Logging_ general** que no dependa de la red
- **_Retries_ personalizados** que se quieran controlar manualmente
- Reescritura de **_requests_ y respuestas** a nivel de aplicación

Comportamiento clave:
- Se ejecuta **una sola vez por _request_**, incluso si hay _redirects_ o _retries_ internos de OkHttp.
- **Ve respuestas provenientes del _cache_**, porque OkHttp puede resolver una _request_ desde disco antes de tocar la red. 
- **No puede modificar la política del _cache_** (qué se guarda, cuándo expira, cómo se revalida), solo puede ver el resultado final. 
- No ve la versión final de la _request_ tal como OkHttp la enviaría por red, porque no participa en las transformaciones de bajo nivel (compresión, _headers_ automáticos, etc.).

Ejemplo:

```kotlin
val okHttpClient = OkHttpClient.Builder()
    .addInterceptor { chain ->
        val newRequest = chain.request()
            .newBuilder()
            .addHeader("User-Agent", "MyApp/1.0")
            .build()

        chain.proceed(newRequest)
    }
```

#### 🌐 *Network Interceptor*
Se ejecuta dos veces por ciclo de red: **_una al enviar la request al servidor_** y **_otra al recibir la respuesta desde la red_**. En resumen: es el **_interceptor para lógica estrictamente de red_**, no para lógica de aplicación.

Es ideal para:
- **_Logging_ real de red** (lo que realmente se envió y lo que realmente llegó)
- Inspeccionar **_headers_ generados por el servidor, _proxies_ o _gateways_** 
- Manipular _headers_ relacionados con **_cache_** (_Cache-Control_, _ETag_, _If-Modified-Since_)
- Operaciones que requieren acceso directo a la **conexión** (certificados, TLS, tamaño real de payload, etc.)

Comportamiento clave:
- Se ejecuta **en cada redirección**, porque cada salto reenvía la _request_ al servidor.
- **No se ejecuta cuando la respuesta proviene del _cache_** :arrow_right: Solo corre cuando hay un acceso real a la red.
- Ve la _request_ **después** de que OkHttp aplicó todas las transformaciones finales (como compresión o _headers_ automáticos).
- Puede modificar la respuesta **antes de que llegue a la capa superior**, lo cual es útil para casos muy específicos (no recomendado para lógica general).

Ejemplo:

```kotlin
val okHttpClient = OkHttpClient.Builder()
    .addNetworkInterceptor { chain ->
        val response = chain.proceed(chain.request())
        // Ideal para depurar headers reales enviados/recibidos
        response
    }
```

### Instancia de Retrofit
Para crear una instancia de Retrofit, es necesario llamar al _builder_ y configurar lo que se requiera. Esto puede hacerse en un archivo separado o como parte de un inyector de dependencias.

Consta de algunos elementos comunes:
- ``baseUrl`` :arrow_right: Configura la URL base de la API a consumir. Debe terminar con la ``/``.
- ``addConverterFactory`` :arrow_right: Agrega un _factory_ que creará una instancia del conversor que permitirá serializar y deserializar objetos. El más habitual suele ser **_Gson_**, pero existen varios más, incluyendo el de **_KotlinX Serialization_**.

```kotlin
val json = Json {
    ignoreUnknownKeys = true
    isLenient = true // Solo si es absolutamente necesario (APIs que no cumplen estándares)
}

Retrofit
    .Builder()
    .baseUrl("https://api.miservicio.com/")
    .addConverterFactory(
        json.asConverterFactory("application/json; charset=UTF-8".toMediaType())
        // También es bastante habitual utilizar el Converter de Gson:
        // GsonConverterFactory.create()
    )
    .client(okHttpClient) // Cliente OkHttp creado en el paso previo
    .build()
    .create(SampleApiService::class.java) // Implementación del servicio
```

### Manejo de respuestas y errores
> ⚠️ Importante:  
> Retrofit **NO lanza excepción** en errores HTTP (4xx/5xx) al usar ``Response<T>``. Solo lanza ``HttpException`` en errores HTTP si el método NO devuelve ``Response<T>``.  
> Retrofit **SÍ lanza excepción** en errores de red (_timeout_, DNS, desconexión, SSL) o serialización (JSON mal formado).

El manejo completo implica distinguir tres niveles:

1. **Excepciones de red o serialización (_throw_)**
Ocurre antes de recibir respuesta (conectividad, timeout, SSL…).

```kotlin
return try {
    val response = api.fetchUser()
    // Pasa al punto 2
    handleResponse(response)
} catch (e: IOException) {
    // Errores de red
    Result.Error("Network error: ${e.localizedMessage}")
} catch (e: MalformedJsonException) {
    // Errores de JSON
    Result.Error("Serialization error: ${e.localizedMessage}")
} catch (e: Exception) {
    // Errores inesperados
    Result.Error("Unexpected error: ${e.localizedMessage}")
}
```

2. **Respuesta HTTP exitosa o con error (2xx / 4xx / 5xx)**
Retrofit devuelve un ``Response<T>``.

```kotlin
fun handleResponse(response: Response<SampleFetchResponse>): Result<SampleFetchResponse> {
    if (response.isSuccessful) {
        val body = response.body()
        return if (body != null) {
            Result.Success(body)
        } else {
            Result.Error("Response body is null")
        }
    } else {
        // Error 4xx / 5xx
        val code = response.code()
        val errorMsg = response.errorBody()?.string()

        return Result.Error("HTTP $code: $errorMsg")
    }
}
```

3. **Mapeo final a un modelo de dominio**
Se puede estandarizar con un _wrapper_ propio, como puede ser ``Result``. Revisar también el uso de [``Either``](/Code%20Snippets%20with%20Kotlin/JSON%20operations%20&%20Error%20handling%20with%20Either.md#sealed-class-either-left-and-right)

```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val message: String) : Result<Nothing>()
}
```

## *Ktor* (cliente)
> 🔍 Referencias:  
> https://ktor.io/  
> https://www.slf4j.org/  
> https://logback.qos.ch/  
> https://logging.apache.org/log4j/2.x/index.html

Ktor es un _framework_ para crear aplicaciones asincrónicas **del lado del servidor y del lado del cliente** con facilidad.  
Incluye un cliente HTTP asincrónico multiplataforma, que permite realizar solicitudes, manejar respuestas y ampliar su funcionalidad con _plugins_, como autenticación, serialización JSON y más.  
A diferencia de Retrofit, Ktor **no usa anotaciones ni interfaces**: se trabaja directamente con un cliente configurado y se realiza cada solicitud mediante la función `client.request{}`. Aunque también cuenta con **funciones de extensión de conveniencia** para los métodos HTTP más comunes (GET, POST, PUT, DELETE).

Para utilizar el cliente HTTP de Ktor en un proyecto Android, se deben configurar los repositorios y agregar las dependencias mandatorias y opcionales en caso de requerirlas.

### 🚀 Cheatsheet Ktor Client
1. **Definir el modelo de datos**

```kotlin
@kotlinx.serialization.Serializable
data class UserDto(
    val id: String,
    val name: String
)
```

2. **Crear instancia de Ktor Client**

```kotlin
val client = HttpClient {
    // Fuerza el lanzamiento de ClientRequestException / ServerResponseException
    expectSuccess = true
    
    install(ContentNegotiation) {
        json() // kotlinx.serialization
    }
    
    install(HttpTimeout) {
        requestTimeoutMillis = 15_000
    }
    
    install(DefaultRequest) {
        url("https://myapi.com/")
        // Función corta directa para agregar un 'header'
        header("User-Agent", "My-App/1.0")
        // Accediendo a la propiedad 'headers' (que es un 'HeadersBuilder').
        headers.appendIfNameAbsent("X-Custom-Header", "Hello")
    }
    
    // Alternativa: Función de extensión idomática (Recomendada por Ktor)
    defaultRequest {
        url("https://api.example.com/")
        header(HttpHeaders.ContentType, "application/json")
    }
}
```

3. **Crear “servicio” (una clase por cada conjunto de _endpoints_)**

```kotlin
class UserApi(private val client: HttpClient) {
    suspend fun fetchUser(id: String): HttpResponse {
        return client.get("users/$id")
    }
}
```

4. **Instanciar el servicio**

```kotlin
val api = UserApi(client)
```

5. **Ejecutar _request_ + Manejo de respuesta y errores**

```kotlin
suspend fun getUser(id: String): Result<UserDto> {
    return try {
        val response = api.fetchUser(id)

        if (response.status.isSuccess()) {
            val body = response.body<UserDto>()
            Result.success(body)
        } else {
            Result.failure(
                Exception("HTTP ${response.status.value}: ${response.status.description}")
            )
        }

    } catch (e: Exception) {
        Result.failure(Exception("Network/serialization error: ${e.localizedMessage}"))
    }
}
```

### Dependencias y permisos
Luego de asegurarse que está agregado el repositorio ``mavenCentral()``, se pueden agregar las dependencias en el archivo ``libs.versions.toml``:

```toml
[versions]
ktor = "{VERSION}"
slf4j = "{VERSION}"

[libraries]
ktor-client-core = { module = "io.ktor:ktor-client-core", version.ref = "ktor" }
ktor-client-okhttp = { module = "io.ktor:ktor-client-okhttp", version.ref = "ktor" }
ktor-client-logging = { module = "io.ktor:ktor-client-logging", version.ref = "ktor" }
ktor-client-content-negotiation = { module = "io.ktor:ktor-client-content-negotiation", version.ref = "ktor" }
ktor-serialization-kotlinx-json = { module = "io.ktor:ktor-serialization-kotlinx-json", version.ref = "ktor" }
slf4j-android = { module = "org.slf4j:slf4j-android", version.ref = "slf4j" }
```

**A tener en cuenta**:

- La funcionalidad principal del cliente está disponible en el artefacto ``ktor-client-core``.
- Un **motor** (**_engine_**) se encarga de **procesar las solicitudes de red**. Existen diferentes motores de cliente disponibles para diversas plataformas, como Apache, CIO, Android, iOS, etc.
- Muchas aplicaciones requieren **funciones comunes que escapan a la lógica de la aplicación**. Estas pueden ser funciones como el _logging_, la serialización o la autorización. Todas estas funciones se proporcionan en Ktor mediante **_plugins_**.
- En JVM, Ktor utiliza **_Simple Logging Facade for Java_** (**_SLF4J_**) como una capa de abstracción para el _logging_. SLF4J desacopla la API de _logging_ de la implementación de _logging_ subyacente, lo que permite integrar el _framework_ de _logging_ que mejor se adapte a los requisitos de la aplicación. Las opciones más comunes incluyen **_Logback_** o **_Log4j_**. Si no se proporciona ningún _framework_, SLF4J utilizará por defecto una implementación sin operación (NOP), que básicamente deshabilita el _logging_.

Implementar las dependencias en el archivo ``build.gradle.kts`` del módulo que corresponda:

```kotlin
// Ktor
implementation(libs.ktor.client.core)
implementation(libs.ktor.client.okhttp)
implementation(libs.ktor.client.logging)
implementation(libs.ktor.client.content.negotiation)
implementation(libs.ktor.serialization.kotlinx.json)
implementation(libs.slf4j.android)
```

Agregar el permiso de internet en el ``Manifest``:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

### Modelo de respuesta con KotlinX Serialization
Ktor sí depende de KotlinX Serialization para el manejo de JSONs, por lo cual se requiere anotar los modelos con ``@Serializable``.

```kotlin
@Serializable
data class UserDto(
    @SerialName("user_id")
    val userId: String,
    @SerialName("name")
    val name: String,
    @SerialName("nickname")
    val nickname: String,
    @SerialName("followers")
    val followers: Int,
    @SerialName("following")
    val following: List<String>,
    @SerialName("user_type")
    val userType: Int,
)
```

### Configuración del cliente HTTP
> ⚠️ Importante:  A diferencia de Retrofit, el cliente de Ktor sí debe cerrarse cuando ya no va a utilizarse.  
> Para eso, se llama a ``client.close()``.
> Si se usa DI esto no hace falta, ya que el cliente **se cierra automáticamente** al liberar el contenedor.

Ktor requiere configurar un **_engine_**: para Android/JVM, el más común es **OkHttp**. En KMP, se suele usar CIO.  
Además, toda la funcionalidad extra se incorpora mediante **_plugins_**, los cuales se agregan con ``install``:

- ``ContentNegotiation`` :arrow_right: JSON via _KotlinX Serialization_
- ``Logging`` :arrow_right: _Logging_ configurable (dependiendo del backend SLF4J)
- ``DefaultRequest`` :arrow_right: URL base, _headers_ comunes, etc.

Ejemplo:

```kotlin
val client = HttpClient(engineFactory = OkHttp) {
    install(plugin = ContentNegotiation) {
        json(
            Json {
                ignoreUnknownKeys = true
                isLenient = false
            }
        )
    }

    if (BuildConfig.DEBUG) {
        install(Logging) {
            logger = Logger.DEFAULT
            level = LogLevel.BODY
        }
    }

    install(plugin = DefaultRequest) {
        url("https://api.miservicio.com/")
        // Función corta directa para agregar un 'header'
        header("User-Agent", "My-App/1.0")
        // Accediendo a la propiedad 'headers' (que es un 'HeadersBuilder').
        headers.appendIfNameAbsent("X-Custom-Header", "Hello")
    }

    // Alternativa: Función de extensión idomática (Recomendada por Ktor)
    defaultRequest {
        url("https://api.example.com/")
        header("User-Agent", "My-App/1.0")
    }
}
```

### Realizar solicitudes
Ktor no utiliza interfaces como Retrofit: se usa la función ``client.request``. Aunque también cuenta con **funciones de extensión de conveniencia** para los métodos HTTP más comunes (GET, POST, PUT, DELETE).

La clase ``HttpRequestBuilder`` ofrece:

- Método HTTP (``method = HttpMethod.Get``)
- URL (``url("users/1")``)
- Headers (``headers.append``)
- Body (``setBody()``)

Ejemplo:

```kotlin
suspend fun fetchUser(client: HttpClient): SampleResponse {
    val response: HttpResponse = client.request {
        method = HttpMethod.Get
        url("users/1")
        header("Journey-Id", "12345")
    }

    // Alternativa con la función de extensión de conveniencia para GET
    val response: HttpResponse = client.get("users/1") {
        header("Journey-Id", "12345")
    }

    return response.body()
}
```

### Manejo de respuestas y errores
> ⚠️ **Importante**:  
> - **Errores de Red y Serialización**: Ktor **SÍ lanza excepción** siempre en fallos de conectividad (_timeout_, DNS, desconexión, errores SSL) o si la deserialización del cuerpo de la respuesta falla. 
> - **Respuestas HTTP no exitosas (4xx / 5xx)**:
>   - Por defecto, la propiedad ``expectSuccess`` está desactivada (``false``). Esto significa que si se pide una respuesta genérica (``val response: HttpResponse = client.get(...)``), Ktor **NO lanzará excepción** al recibir un _status_ 4xx o 5xx; simplemente se obtendrá el ``HttpResponse`` con su respectivo _status code_. 
>   - **Excepción**: Si se deserializa directamente la respuesta (``val user: UserDto = client.get(...).body()``), Ktor **SÍ lanzará excepción** (``ClientRequestException`` para 4xx, ``ServerResponseException`` para 5xx) si la respuesta no es exitosa. 
>   - **Configuración global**: Si se desea que Ktor lance automáticamente excepciones 4xx/5xx en todas las solicitudes sin importar cómo se consuman, se puede activar ``expectSuccess = true`` en la configuración del cliente o personalizar la validación mediante ``HttpResponseValidator {}``.

El tipo de respuesta que devuelve es un ``HttpResponse``.

Ejemplo:

```kotlin
val client = HttpClient {
    // Fuerza el lanzamiento de ClientRequestException / ServerResponseException
    expectSuccess = true
    
    // Resto de la configuración...
}

suspend fun safeCall(client: HttpClient): Result<SampleResponse> {
    return try {
        val response: HttpResponse = client.request {
            url("users/1")
        }

        if (response.status.isSuccess()) {
            Result.Success(response.body())
        } else {
            Result.Error("HTTP ${response.status.value}: ${response.bodyAsText()}")
        }

    } catch (e: Exception) {
        Result.Error("Network error: ${e.localizedMessage}")
    }
}
```

## *Sentry*
> 🔍 Referencia:  
> https://sentry.io/welcome/

Es una plataforma de _software_ de **código abierto** y un **servicio alojado (SaaS)** diseñado para ayudar a los desarrolladores a **rastrear, monitorear y resolver errores y problemas de rendimiento en sus aplicaciones en tiempo real**.

### Características Principales
- **Monitoreo de Errores (_Error Tracking_):** Captura automáticamente excepciones no controladas (*uncaught exceptions*), fallos (*crashes*) y otros errores a medida que ocurren en la aplicación.
- **Monitoreo del Rendimiento (APM):** Permite medir métricas clave, identificar cuellos de botella y analizar transacciones lentas (como cargas de página o llamadas a API), proporcionando un seguimiento distribuido a través de todo el *stack* de la aplicación.
- **Contexto Detallado:** Adjunta información valiosa a cada error o evento de rendimiento, incluyendo *stack traces* completos, estado del dispositivo (OS, memoria, batería), acciones del usuario ("*breadcrumbs*" o "migas de pan"), y el *commit* exacto que pudo introducir el error.
- **Alertas en Tiempo Real:** Notifica a los equipos de desarrollo instantáneamente a través de herramientas de colaboración como Slack, GitHub o Jira cuando surgen nuevos problemas o regresiones.

### Cómo se integra con Android
La integración de Sentry con una aplicación Android se logra principalmente a través del uso de su **SDK nativo para Android** (compatible con Kotlin y Java), el cual se integra en el sistema de construcción de la aplicación (Gradle).  
Una vez integrado, el SDK escucha automáticamente los fallos y errores, y los reporta al _dashboard_ centralizado de Sentry, proporcionando un contexto completo para identificar y resolver el problema rápidamente.

#### 1. Configuración en la Plataforma Sentry
Se crea un proyecto de tipo Android en la interfaz de Sentry. Esto genera una clave de cliente única llamada **DSN** (**_Data Source Name_**), que es esencial para conectar la app con el servidor de Sentry.

#### 2. Adición de Dependencias Gradle
Se añaden las dependencias del SDK de Sentry al archivo `build.gradle` (o `build.gradle.kts`) de la aplicación Android.

📌 Ejemplo:

```kotlin
dependencies {
    implementation("io.sentry:sentry-android:7.15.0")
}
```

#### 3. Inicialización del SDK
El SDK se inicializa con el DSN en el código de la aplicación, generalmente en la clase `Application` o la actividad principal.

📌 Ejemplo:

1. En la clase que hereda de ``Application``
```kotlin
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()

        Sentry.init { options ->
            options.dsn = "https://TU_DSN_AQUI.ingest.sentry.io/123456"
            options.tracesSampleRate = 1.0  // (Opcional) habilita performance monitoring
        }
    }
}
```

2. En el ``Manifest``
```xml
<application
    android:name=".MyApp"
    ... >
</application>
```

#### 4. Integración con *Gradle Plugin*
Para aplicaciones Android ofuscadas con **_ProGuard o R8_**, Sentry proporciona un *plugin* de Gradle que automatiza la carga de los archivos de mapeo (*mapping files*) al servidor de Sentry durante el proceso de CI/CD. Esto es crucial para **desofuscar** los *stack traces* y hacerlos legibles para los desarrolladores.  
Este plugin se agrega en el archivo ``build.gradle.kts(root)``.

📌 Ejemplo:

```kotlin
plugins {
    id("io.sentry.android.gradle") version "4.9.0"
}
```

#### 5. Monitoreo NDK (Opcional)
Sentry también ofrece integración **NDK** (**_Native Development Kit_**) para capturar fallos que ocurren en código C/C++ nativo utilizado en la aplicación.

#### 6. Uso del Asistente (*Sentry Wizard*)
Para simplificar el proceso, Sentry ofrece una herramienta de línea de comandos (`sentry-wizard`) que puede automatizar la mayoría de estos cambios de configuración en el proyecto Android.

## *Segment*
> 🔍 Referencia:  
> https://segment.com/

Es una **Plataforma de Datos de Clientes** (**CDP**, por sus siglas en inglés) cuyo propósito principal es **_recopilar, unificar, gobernar y enrutar datos de clientes de múltiples fuentes_** (sitios web, aplicaciones móviles, servidores _backend_, etc.) a cientos de herramientas de análisis, _marketing_ y almacenamiento de datos, todo con una única implementación de código.

En esencia, Segment **_resuelve el problema de los "silos de datos"_**, permitiendo a las empresas tener una "vista única y completa del cliente" (perfil de cliente 360 grados) para potenciar la personalización, la segmentación de audiencia y la toma de decisiones basada en datos. Esta arquitectura permite a los equipos de ingeniería implementar el seguimiento de datos **_una sola vez_**, mientras que los equipos de negocio pueden experimentar y añadir nuevas herramientas de análisis o _marketing_ libremente.

### Características Principales
- **Recopilación Centralizada:** Utiliza una API o SDKs para capturar datos de eventos (acciones del usuario, rasgos de usuario, etc.) de manera uniforme en todas las plataformas.
- **Unificación de Identidades (_Identity Resolution_):** Combina datos de un mismo usuario provenientes de diferentes puntos de contacto (por ejemplo, su actividad en la web y su actividad en la app Android) en un solo perfil coherente.
- **Gestión de Esquemas (_Schema Management_):** Ayuda a los equipos a definir y gobernar la estructura de los datos que están rastreando, asegurando la consistencia y calidad de los datos.
- **Activación de Datos (_Data Activation_):** Envía los datos unificados a más de 200 herramientas asociadas (_Google Analytics_, _Mixpanel_, _Salesforce_, _Sentry_, plataformas publicitarias, etc.) con solo "pulsar un interruptor", sin necesidad de escribir código adicional para cada integración individual.

### Cómo se integra con Android
El proceso se realiza mediante el uso del **SDK nativo de Segment para Android** (actualmente, el [**Analytics-Kotlin SDK**](https://segment.com/docs/connections/sources/catalog/libraries/mobile/kotlin-android/) es el recomendado para nuevos proyectos).

#### 1. Configuración del Origen (*Source*) en Segment
En el panel de control de _Twilio Segment_, se configura una nueva "**Fuente**" (**_Source_**) de tipo "**_Kotlin (Android)_**". Esto proporciona una clave de escritura (`Write Key`) única para la aplicación.

#### 2. Integración del SDK en Gradle
El SDK se añade como una dependencia en el archivo `build.gradle` o `build.gradle.kts` del proyecto Android, la cual se descarga desde _Maven Central_.

📌 Ejemplo:

```kotlin
dependencies {
    implementation("com.segment.analytics.kotlin:android:1.11.7")
}
```

#### 3. Inicialización en la App
El SDK se inicializa en la aplicación Android con la `Write Key` obtenida en el paso 1.

📌 Ejemplo:

1. En la clase que hereda de ``Application``
```kotlin
import android.app.Application
import com.segment.analytics.kotlin.android.Analytics
import com.segment.analytics.kotlin.core.Analytics as AnalyticsCore

class MyApp : Application() {

    override fun onCreate() {
        super.onCreate()

        Analytics(this) {
            writeKey = "YOUR_WRITE_KEY"
            trackApplicationLifecycleEvents = true
            collectDeviceId = true
        }

        // Opcional, útil para desarrollo
        AnalyticsCore.debug = true
    }
}
```

2. En el ``Manifest``
```xml
<application
    android:name=".MyApp"
    ... >
</application>
```

#### 4. Implementación de Seguimiento (*Tracking*)
Se añaden llamadas específicas a la API de Segment en puntos clave de la aplicación para rastrear eventos y propiedades del usuario. Los tres métodos principales son:
- `track()`: Para registrar acciones que el usuario realiza (ej. "Producto Visto", "Pedido Completado").
- `identify()`: Para asociar acciones con un usuario específico y registrar sus rasgos (ej. nombre, correo electrónico, plan de suscripción).
- `screen()`: Para registrar qué pantallas ha visitado el usuario dentro de la app.

📌 Ejemplos:

1. ``track()`` — Registrar acciones del usuario
```kotlin
import com.segment.analytics.kotlin.android.Analytics

fun onProductViewed(productId: String, productName: String) {
    Analytics.track(
        event = "Producto Visto",
        properties = buildJsonObject {
            put("id", productId)
            put("nombre", productName)
        }
    )
}
```

2. ``identify()`` — Identificar al usuario + rasgos
```kotlin
import com.segment.analytics.kotlin.android.Analytics

fun identifyUser(userId: String, email: String, name: String) {
    Analytics.identify(
        userId = userId,
        traits = buildJsonObject {
            put("email", email)
            put("name", name)
            put("plan", "premium")
        }
    )
}
```

3. ``screen()`` — Registrar pantallas visitadas
```kotlin
import com.segment.analytics.kotlin.android.Analytics

fun trackScreenHome() {
    Analytics.screen(
        screenName = "Home",
        properties = buildJsonObject {
            put("seccion_destacada", true)
        }
    )
}
```

#### 5. Activación de Destinos (*Destinations*)
Una vez que los datos fluyen de la app a Segment, el equipo de _marketing_ o producto puede activar integraciones con otras herramientas (ej. enviar todos los eventos de "Pedido Completado" a _Google Ads_ o a un *data warehouse*) simplemente configurándolo en la interfaz web de Segment, sin cambios en el código de la app.
