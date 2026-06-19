2do Parcial - Parte Practica
Que se solicita:

El codigo tiene 10 errores. Recae en usted analizar que es un error dentro del codigo.
Los Alumnos tendran que forkear este repo como propio, hacer un issue desde Github con Comentarios refiriendo en que linea esta el error, y como se debe solucionar.
La respuesta sera con el link a ese Fork, y adentro deben estar los issues. Los profesores tenemos que poder ingresar al mismo. Recae en los alumnos asegurarse de que los profesores puedan ingresar.
Tambien pueden editar el Archivo Readme y poner los resultados dentro de sus propios forks.
https://github.com/ExBattou/SimpsonsApp


---

## Errores encontrados

### Error 1
**Archivo:** `main/MainScreen.kt` línea 51
Tenesmo este bloque de codigo en MainScreen.kt
```kotlin
if (episodes.loadState.refresh is LoadState.NotLoading && seasons.isEmpty()) {
    viewModel.refreshSeasons()
}
```
Esta responsabilidad le corresponde a el ViewModel no a composable ya que la decision de cargar o no mas season es un logica de negocio y el composable solo se debe centrar en mostras nada mas. 
**Solución:** Migrar esta logica al ViewModel

### Error 2 
**Archivo:** `main/MainViewModel.kt` y `main/MainScreenViewModel.kt`
`MainScreenViewModel` es un ViewModel creado para tests que nunca fue eliminado. El código de test no le pertenece a la capa de presentación de la app.

**Solución:** Mover `MainScreenViewModel` al directorio de tests donde corresponde.

### Error 3
**Archivo:** `data/remote/EpisodeRemoteMediator.kt` línea 105
```kotlin
interface SimpsonsApi {
    @GET("https://thesimpsonsapi.com/api/episodes")
    suspend fun getEpisodes(@Query("page") page: Int): EpisodesResponse
}
```
Definir esta interfaz no es responsabilidad del `RemoteMediator`. `SimpsonsApi` debería estar dentro de `data/remote/`.

**Solución:** Crear un archivo `SimpsonsApi.kt` dentro de `data/remote/` y mover la interfaz allí.

### Error 4
**Archivo:** `data/repository/EpisodeRepositoryImpl.kt` línea 53
```kotlin
fun EpisodeEntity.toDomain() = Episode(id = id, airdate = airdate, ...)
```
El `toDomain()` debería estar en el propio archivo de `EpisodeEntity` o ne us defecto con un mapper.

**Solución:** Crear un archivo `EpisodeMapper.kt` en `data/local/entity/` y mover la función allí.

### Error 5
**Archivo:** `data/DataRepository.kt`
```kotlin
interface DataRepository {
    val data: Flow<List<String>>
}
```
Esta interfaz es para test unicamente.

**Solución:** Definir el fake directamente dentro del directorio de tests.

### Error 6
**Archivos:** `domain/usecase/GetEpisodesUseCase.kt` y demás use cases
```kotlin
operator fun invoke() = repository.get_episodes()
```
Los Use Case se deben usar en caso de que se quiere aplicar logica de negocio, en este caso el metodo solo llama a get_episodes por lo tanto su existencia no tiene sentido, no romper pero tampoco aporta.

**Solución:** Eliminarlos y llamar al repositorio directamente desde el ViewModel.

### Error 7
**Archivo:** `data/remote/EpisodeRemoteMediator.kt` línea 106
```kotlin
@GET("https://thesimpsonsapi.com/api/episodes")
```
Definir la URL base no es responsabilidad de la anotación del endpoint. Le pertenece al `Retrofit.Builder()` en `DataModule`, donde puede cambiarse por entorno sin tocar el código de red.

**Solución:** Agregar `.baseUrl("https://thesimpsonsapi.com/")` en `DataModule` y dejar solo el path relativo `"api/episodes"` en la anotación `@GET`.

### Error 8
**Archivo:** `domain/repository/EpisodeRepository.kt` línea 8
```kotlin
fun get_episodes(): Flow<PagingData<Episode>>
```
Respetar las convenciones del lenguaje es responsabilidad de quien define el contrato. `get_episodes()` usa snake_case, que no pertenece a Kotlin. La implementación lo define como `getEpisodes()`, rompiendo el contrato de la interfaz.

**Solución:** Renombrar `get_episodes()` a `getEpisodes()` en la interfaz para que coincida con la implementación y respete las convenciones de Kotlin.

### Error 9
**Archivo:** `main/MainScreen.kt` línea 119
```kotlin
LazyRow(state = listState, modifier = Modifier.fillMaxSize()) { ... }
```
Decidir la dirección de scroll de una lista de contenido no es responsabilidad del Composable. Una lista de episodios con imagen, título y sinopsis debería usar `LazyColumn` (scroll vertical), no `LazyRow`.

**Solución:** Reemplazar `LazyRow` por `LazyColumn`.

### Error 10
**Archivo:** `di/DataModule.kt` línea 27
```kotlin
level = HttpLoggingInterceptor.Level.BODY
```
Controlar qué se loguea según el entorno no es responsabilidad del interceptor, sino de la configuración del módulo.

**Solución:** Envolver la creación del interceptor con `if (BuildConfig.DEBUG)` y usar nivel `NONE` en producción.
