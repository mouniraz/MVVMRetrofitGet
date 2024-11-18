# MVVMRetrofitGet
This workshop is to create repositories in data layer and inject dependencies. To do we will start with the code of the application get api on the git repo https://github.com/mouniraz/ConsumeApiRetrofit-part1- 
(if you dont have code you can pull from master branch) 
### N.B
Base URL has changed to https://tyradex.vercel.app/api/v1/ 
# Step1: Create Data Layer 
First, you need to create the repository class. The Android developers guide states that repository classes are named after the data they're responsible for. The repository naming convention is type of data + Repository. In your app, this is PokemonRepository.

### Create repository
1. Right-click on com.example.yourapp and select New > Package.
2. In the dialog, enter data.
3. Right-click on the data package and select New > Kotlin Class/File.
4. In the dialog, select Interface and enter MarsPhotosRepository as the name of the interface.
5. Inside the PokemonRepository interface, add an abstract function called getAllPokemon(), which returns a list of MPokemon objects. It is called from a coroutine, so declare it with suspend.
```kotlin
interface PokemonRepository {
    suspend fun  getAllPokemon():List<PockemonsItem>
}
```
6. Below the interface declaration, create a class named NetworkPokemonRepository for implementing the PokemonRepository interface.
7. Inside the PokemonRepository class, override the abstract function getAllPokemon(). This function returns the data from calling MarsApi.ApiService.ApiService()
```kotlin
interface PokemonRepository {
    suspend fun  getAllPokemon():List<PockemonsItem>
}
class NetworkPokemonRepository:PokemonRepository{
    override suspend fun getAllPokemon(): List<PockemonsItem> {
        return PockApi.retrofitService.getAllPokemon()
    }

}
```
7. Next, you need to update the ViewModel code to use the repository to get the data as Android best practices suggest.

Open the viewmodel/ViewModel.kt file.
Scroll down to the getAllPokemon() method.
Replace the line "val listResult = PockApi.retrofitService.getAllPokemon()" with the following code:
```kotlin
   val pokemonRepository = NetworkPokemonRepository()
   val listResult = pokemonRepository.getAllPokemon()
```
8. Run the app. Notice that the results displayed are the same as the previous results.
Instead of the ViewModel directly making the network request for the data, the repository provides the data. The ViewModel no longer directly references the MarsApi code.
# Step2: Dependency Injection
### Create an Application Container
1. Right-click on the data package and select New > Kotlin Class/File.
2. In the dialog, select Interface, and enter AppContainer as the name of the interface.
3. Inside the AppContainer interface, add an abstract property called pokemonRepository of type PokemonRepository. 

```kotlin
interface AppContainer {
    val pokemonRepository:PokemonRepository
}
```
4. Below the interface definition, create a class called DefaultAppContainer that implements the interface AppContainer.
5. From network/ApiService.kt, move the code for variables BASE_URL, retrofit, and retrofitService into the DefaultAppContainer class so that they are all located within the container that maintains the dependencies.
```kotlin
class DefaultAppContainer : AppContainer {
    
    private val BASE_URL =
        "https://android-kotlin-fun-mars-server.appspot.com"

    private val retrofit: Retrofit = Retrofit.Builder()
        .addConverterFactory(GsonConverterFactory.create()) // For JSON parsing
        .baseUrl(BASE_URL)
        .build()

    val retrofitService: ApiService by lazy {
        retrofit.create(ApiService::class.java)
    }


}
```

6.override val pokemonRepository as below
```kotlin
override val pokemonRepository: PokemonRepository by lazy {
        NetworkPokemonRepository(retrofitService)
    }
```
7. the code of AppContainer is like this
```kotlin
  
interface AppContainer {
    val pokemonRepository:PokemonRepository

}

class DefaultAppContainer : AppContainer {
    override val pokemonRepository: PokemonRepository by lazy {
        NetworkPokemonRepository(retrofitService)
    }
    private val BASE_URL =
        "https://android-kotlin-fun-mars-server.appspot.com"

    private val retrofit: Retrofit = Retrofit.Builder()
        .addConverterFactory(GsonConverterFactory.create()) // For JSON parsing
        .baseUrl(BASE_URL)
        .build()

    val retrofitService: ApiService by lazy {
        retrofit.create(ApiService::class.java)
    }


} 
```
8. Open the data/PokemonRepository.kt file. We are now passing retrofitService to NetworkPokemonRepository, and you need to modify the NetworkPokemonRepository class.
9. In the NetworkPokemonRepository class declaration, add the constructor parameter pokemonApiService as shown in the following code.
```kotlin
class NetworkPokemonRepository(private val pokemonApiService: ApiService
):PokemonRepository{
    override suspend fun getAllPokemon(): List<PockemonsItem> {
        return pokemonApiService.getAllPokemon()
    }

}
```
10. In the NetworkPokemonRepository class, in the getAllPokemon() function, change the return statement to retrieve data from PokemonApiService.
```kotlin
class NetworkPokemonRepository(private val pokemonApiService: ApiService
):PokemonRepository{
    override suspend fun getAllPokemon(): List<PockemonsItem> {
        return pokemonApiService.getAllPokemon()
    }

}
```
11. remove
```kotlin    
object PockApi {
    }
```
