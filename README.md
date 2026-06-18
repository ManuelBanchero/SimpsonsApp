2do Parcial - Parte Practica
Que se solicita:

El codigo tiene 10 errores. Recae en usted analizar que es un error dentro del codigo.
Los Alumnos tendran que forkear este repo como propio, hacer un issue desde Github con Comentarios refiriendo en que linea esta el error, y como se debe solucionar.
La respuesta sera con el link a ese Fork, y adentro deben estar los issues. Los profesores tenemos que poder ingresar al mismo. Recae en los alumnos asegurarse de que los profesores puedan ingresar.
Tambien pueden editar el Archivo Readme y poner los resultados dentro de sus propios forks.
https://github.com/ExBattou/SimpsonsApp

---

## Errores Encontrados

### Error 1: Problemas de Nomenclatura y Convenciones Kotlin

#### 1.1: Inconsistencia de nomenclatura - EpisodeRepository.kt (Línea 8)
**Archivo:** `app/src/main/java/com/example/simpsonsapp/domain/repository/EpisodeRepository.kt`
**Problema:** El método está declarado como `get_episodes()` usando snake_case, violando la convención de nombres de Kotlin que requiere camelCase.
```kotlin
fun get_episodes(): Flow<PagingData<Episode>>
```
**Solución:** Cambiar a camelCase: `fun getEpisodes(): Flow<PagingData<Episode>>`

#### 1.2: Inconsistencia en llamada de método - GetEpisodesUseCase.kt (Línea 13)
**Archivo:** `app/src/main/java/com/example/simpsonsapp/domain/usecase/GetEpisodesUseCase.kt`
**Problema:** Llama a `repository.get_episodes()` (snake_case) que no coincide con la implementación en EpisodeRepositoryImpl.
```kotlin
return repository.get_episodes()
```
**Solución:** Cambiar a: `return repository.getEpisodes()` (Esta es una consecuencia directa del Error 1)

---

### Error 2: Problemas de Configuración Retrofit

#### 2.1: URL completa en anotación @GET - EpisodeRemoteMediator.kt (Línea 106)
**Archivo:** `app/src/main/java/com/example/simpsonsapp/data/remote/EpisodeRemoteMediator.kt`
**Problema:** La URL completa está en la anotación @GET. Debería ser un endpoint relativo, con la URL base en Retrofit.
```kotlin
@GET("https://thesimpsonsapi.com/api/episodes")
```
**Solución:** Cambiar a: `@GET("api/episodes")` y configurar baseUrl en Retrofit Builder.

#### 2.2: BaseUrl no configurada - DataModule.kt (Línea 34-37)
**Archivo:** `app/src/main/java/com/example/simpsonsapp/di/DataModule.kt`
**Problema:** El Retrofit Builder no tiene .baseUrl() configurado, lo que causará error en runtime. (Conectado al Error 3)
```kotlin
return Retrofit.Builder()
    .client(client)
    .addConverterFactory(GsonConverterFactory.create())
    .build()
```
**Solución:** Agregar `.baseUrl("https://thesimpsonsapi.com/")` antes de `.build()`

---

### Error 3: Problemas de UI y Composables

#### 3.1: Lambda vacía innecesaria - DetailScreen.kt (Línea 60)
**Archivo:** `app/src/main/java/com/example/simpsonsapp/detail/DetailScreen.kt`
**Problema:** Parámetro onClick pasa una lambda vacía que no hace nada en modo detalle.
```kotlin
onClick = { }
```
**Solución:** Remover el parámetro `onClick` de la llamada a EpisodeCard o pasar una lambda apropiada. En modo detalle no debería ser clickeable.

#### 3.2: Componente incorrecto - MainScreen.kt (Línea 119)
**Archivo:** `app/src/main/java/com/example/simpsonsapp/main/MainScreen.kt`
**Problema:** Está usando LazyRow (scroll horizontal) cuando debería ser LazyColumn (scroll vertical) para una lista de episodios.
```kotlin
LazyRow(
    state = listState,
    modifier = Modifier.fillMaxSize()
)
```
**Solución:** Cambiar a `LazyColumn` para un comportamiento de scroll vertical correcto.

#### 3.3: Non-null assertion operator innecesario - DetailScreen.kt (Línea 59)
**Archivo:** `app/src/main/java/com/example/simpsonsapp/detail/DetailScreen.kt`
**Problema:** Uso de `!!` (non-null assertion) en una variable nullable. Si `episode` es null, causará un crash en runtime.
```kotlin
episode = episode!!,
```
**Solución:** Usar el binding seguro o validar que episode no sea null antes. Ya hay validación en el if anterior (línea 51), cambiar a:
```kotlin
if (episode != null) {
    EpisodeCard(
        episode = episode,  // Remover !!
        ...
    )
}
```

---

### Error 4: Problemas de Síntaxis y Estructura del Código

#### 4.1: Código inválido fuera de data class - Episode.kt (Líneas 13-14)
**Archivo:** `app/src/main/java/com/example/simpsonsapp/domain/model/Episode.kt`
**Problema:** Hay un bloque `init` declarado fuera de la data class con un `return` inválido. Esto causará error de compilación.
```kotlin
init {
    return Episode; //NO BORRAR
}
```
**Solución:** Eliminar completamente las líneas 13-14. El código `init` y `return` son inválidos en Kotlin fuera de una clase.

#### 4.2: Wildcard imports innecesarios - AppNavigation.kt (Líneas 9-10)
**Archivo:** `app/src/main/java/com/example/simpsonsapp/AppNavigation.kt`
**Problema:** Importaciones wildcard que no se utilizan adecuadamente. Viola buenas prácticas de limpieza de código.
```kotlin
import androidx.navigation.*
import androidx.compose.*
```
**Solución:** Remover estas líneas y usar importaciones específicas según sea necesario.

---

### Error 5: Problemas de Base de Datos

#### 5.1: Nombre de columna incorrecto - RemoteKeyDao.kt (Línea 14)
**Archivo:** `app/src/main/java/com/example/simpsonsapp/data/local/dao/RemoteKeyDao.kt`
**Problema:** El nombre de columna en la query SQL es "episodeId" pero Room mapea nombres con snake_case para las columnas.
```kotlin
@Query("SELECT * FROM remote_keys WHERE episodeId = :id")
```
**Solución:** Cambiar a snake_case: `@Query("SELECT * FROM remote_keys WHERE episode_id = :id")`

---

### Error 6: Problemas de Arquitectura y Error Handling

#### 6.1: Falta manejo de errores - DetailViewModel.kt (Línea 32-34)
**Archivo:** `app/src/main/java/com/example/simpsonsapp/detail/DetailViewModel.kt`
**Problema:** No hay manejo de excepciones cuando el use case falla. La arquitectura MVVM + Clean Architecture requiere manejo robusto de errores.
```kotlin
private fun loadEpisodeDetail(id: Int) {
    viewModelScope.launch {
        _episode.value = getEpisodeDetailUseCase(id)
    }
}
```
**Solución:** Agregar un StateFlow para estado de error y try-catch para manejar excepciones:
```kotlin
private val _error = MutableStateFlow<Exception?>(null)
val error: StateFlow<Exception?> = _error.asStateFlow()

private fun loadEpisodeDetail(id: Int) {
    viewModelScope.launch {
        try {
            _episode.value = getEpisodeDetailUseCase(id)
        } catch (e: Exception) {
            _error.value = e
        }
    }
}
```

---

### Error 7: Problemas de Configuración del Proyecto

#### 7.1: Ruta hardcodeada de JDK - gradle.properties (Línea 32)
**Archivo:** `gradle.properties`
**Problema:** La línea tiene una ruta absoluta hardcodeada de Java/JDK que es específica de la máquina del profesor/creador original. Esto rompe el build en otras máquinas e impide que se comparta el proyecto.
```properties
#org.gradle.java.home=/opt/homebrew/Cellar/openjdk@17/17.0.15/libexec/openjdk.jdk/Contents/Home
```
**Solución:** Comentar o remover esta línea completamente. Gradle debería autodetectar el JDK del sistema. Solo configurar `org.gradle.java.home` si es absolutamente necesario y con una ruta relativa o variable de entorno, nunca una ruta hardcodeada específica de una máquina.

#### 7.2: Componente muerto (Dead Code) - MainScreenViewModel.kt (Línea 12)
**Archivo:** `app/src/main/java/com/example/simpsonsapp/main/MainScreenViewModel.kt`
**Problema:** MainScreenViewModel existe pero nunca se usa en el proyecto. No tiene @HiltViewModel y no está importado en ningún lugar. Es código muerto que genera confusión.
```kotlin
class MainScreenViewModel(dataRepository: DataRepository) : ViewModel() {
  val aux: StateFlow<MainScreenUiState> = ...
}
```
**Solución:** Eliminar completamente el archivo MainScreenViewModel.kt ya que la funcionalidad equivalente (y correcta) está en MainViewModel.kt. Si se mantiene, debería estar adecuadamente integrado con @HiltViewModel y @Inject.

---

### Error 8: Problemas de Efectos Secundarios en Composables

#### 8.1: Side effect en composable sin LaunchedEffect - MainScreen.kt (Línea 51-52)
**Archivo:** `app/src/main/java/com/example/simpsonsapp/main/MainScreen.kt`
**Problema:** Ejecutar lógica de negocio (viewModel.refreshSeasons()) directamente en el cuerpo del composable. Esto causa que se ejecute en cada recomposición, violando principios de Compose. 
```kotlin
if (episodes.loadState.refresh is LoadState.NotLoading && seasons.isEmpty()) {
    viewModel.refreshSeasons()  // Se ejecuta en cada recomposición
}
```
**Solución:** Usar LaunchedEffect o SideEffect para controlar cuándo se ejecuta:
```kotlin
LaunchedEffect(episodes.loadState.refresh, seasons) {
    if (episodes.loadState.refresh is LoadState.NotLoading && seasons.isEmpty()) {
        viewModel.refreshSeasons()
    }
}
```
