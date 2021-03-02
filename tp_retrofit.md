# Requête HTTP avec Retrofit
Dans ce TP, nous allons expérimenter l'usage de la librairie Retrofit pour consommer les actions d'une API publique de gestion de news. 

## Fonctionnalités du projet
- Afficher une liste de news
- Afficher les détails d'une news
- Afficher le texte complet d'une news dans un navigateur externe
- Partager une news
- Ajouter une news en favoris 

### Contraintes
- Si une connexion internet est disponible, la liste des articles est récupérée sur internet
- Si pas de connexion internet, récupérer la liste des articles dans une base de données locales (gérée avec Room)
- Quand les articles sont récupérés sur internet, les sauvegarder en BDD 

## Rappel du schéma d'architecture
![Ajoutez un voisin](/archi_retrofit.png "Nouveau voisin")

## Créer un nouveau projet 
- Dans Android Studio, créez un nouveau projet ``Newsletter``
- Assurez-vous que le nom de package est unique
- Selectionnez Empty Activity comme template 
- Choisir Kotlin comme langage
- Activer la gestion de version
- Commiter les premiers fichers

## Mise en place du service 
Dans cette section, nous allons mettre en place le service permettant de récupérer les données depuis l'API https://newsapi.org/


### Configuration 
- Ajoutez la permission internet dans le manifest

- Ajoutez les dépendances aux librairies Retrofit, OkHTTP et Gson
```gradle
//Retrofit : Pour consommer les web services RestFul
implementation "com.squareup.retrofit2:retrofit:2.6.0"
//okhttp : Effectuer les requêtes HTTP (utilisée par Retrofit)
implementation "com.squareup.okhttp3:okhttp:3.12.0"
//Gson: Convertir les Json en objet POJ
implementation "com.google.code.gson:gson:2.8.5"
//Gson Converter : Utilisé par Retrofit pour convertir les objets JSON en POJO
implementation "com.squareup.retrofit2:converter-gson:2.4.0"
//Retrofit Logger: Permet d'arricher les logs des requêtes
implementation "com.squareup.okhttp3:logging-interceptor:4.2.1"
//Coroutines : Pour faire du multithreading
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.7"
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.7"
implementation "androidx.lifecycle:lifecycle-runtime-ktx:2.2.0"
```
> Mettez à jour les numéros de versions des librairies s'il le faut 

## Architecture du code
- Créer tous les packages nécessaires en respectant l'architecture défini (inspirez-vous du [TP sur Room](./tp_room.md))

- Pour la gestion des appels au web service, créer un sous-package ```online``` dans le package ```dal``` 

- Ajouter dans le package ```dal``` une interface ```NewsDataSource```

- Ajouter une methode ```getArticle()``` dans ```NewsDataSource``` dont le type de retour est ```LiveData<List<Article>>```

## Récupérer les articles via retrofit

- Créer une data classe qui modélise la réponse du web service
```JSON
{
  "status": "ok",
  "totalResults": 8932,
  "articles": [
    {
      "source": {
        "id": "the-hindu",
        "name": "The Hindu"
      },
      "author": "Reuters",
      "title": "Goldman Sachs restarts cryptocurrency desk amid bitcoin boom",
      "description": "The desk is part of Goldman's activities within the fast-growing digital assets sector, which also includes projects involving blockchain technology and central bank digital currencies.",
      "url": "https://www.thehindu.com/sci-tech/technology/goldman-sachs-restarts-cryptocurrency-desk-amid-bitcoin-boom/article33969292.ece",
      "urlToImage": "https://www.thehindu.com/sci-tech/technology/mq6j54/article33969615.ece/ALTERNATES/LANDSCAPE_615/goldman-sachs-cryptojpg",
      "publishedAt": "2021-03-02T06:21:04Z",
      "content": "(Subscribe to our Today's Cache newsletter for a quick snapshot of top 5 tech stories. Click here to subscribe for free.)\r\nGoldman Sachs Group Inc has restarted its cryptocurrency trading desk and wi… [+2336 chars]"
    },
    ....
  ]
},
``` 
> Comment faire ?
- Créer un sous-package ```model``` dans le package ```online```. 
- Dans le package ```model```, ajoutez un fichier ```Response.kt```
- Ajouter les classes permettant de modéliser la réponse du web service
```kotlin
data class ArticleResponse(
val status: String, 
val totalResults: Int, 
val articles: List<ArticleItem>
)

data class ArticleItem(
val author: String, 
val title: String, 
val description: String,
/**Ajouter les autres attributs **/
)
```
- Créer un package ```service``` dans ```online``` et ajouter une interface ```RetrofitApiService```
```kotlin
interface RetrofitApiService {
    //GET --> On lance une requête de type GET
    // everything est l'action du web service qu'on veut apeler
    // Elle sera concaténée avec l'url prédéfini dans retrofit 
    @GET("/everything")
    fun list(): Call<List<ArticleResponse>>
}
```
> Retrofit utilise aussi des interfaces pour modéliser les différentes actions d'une API REST.

- Implémenter la classe qui modélise un objet métier article dans l'application. Ajouter cette classe dans le package ```models```. 

```Kotlin 
data class Article(
val author: String, 
val title: String, 
val description: String,
/**Ajouter les autres attributs **/
)
```

- Dans le package ```online```, ajouter une implémentation de l'interface ``NewsDataSource``, que vous pouvez appeler ``ArticleOnlineDataSource`` pour récupérer les articles depuis le web serivice 

```kotlin

class ArticleOnlineService : NewsDataSource {
    private val service: RetrofitApiService

    init {
        val retrofit = buildClient()
        //Init the api service with retrofit
        service = retrofit.create(RetrofitApiService::class.java)
    }

    /**
     * Configure retrofit
     */
    private fun buildClient(): Retrofit {
        val httpClient = OkHttpClient.Builder().apply {
            addLogInterceptor(this)
            addApiInterceptor(this)
        }.build()
        return Retrofit
            .Builder()
            .baseUrl(apiUrl)
            .addConverterFactory(GsonConverterFactory.create())
            .client(httpClient)
            .build()
    }

    /**
     * Add a logger to the client so that we log every request
     */
    private fun addLogInterceptor(builder: OkHttpClient.Builder) {
        val httpLoggingInterceptor = HttpLoggingInterceptor().apply {
            level = HttpLoggingInterceptor.Level.BODY
        }
        builder.addNetworkInterceptor(httpLoggingInterceptor)
    }

    /**
     * Intercept every request to the API and automatically add
     * the api key as query parameter
     */
    private fun addApiInterceptor(builder: OkHttpClient.Builder) {
        builder.addInterceptor(object : Interceptor {
            override fun intercept(chain: Interceptor.Chain): Response {
                val original = chain.request()
                val originalHttpUrl = original.url
                val url = originalHttpUrl.newBuilder()
                    .addQueryParameter("apikey", apiKey)
                    .build()

                val requestBuilder = original.newBuilder()
                    .url(url)
                val request = requestBuilder.build()
                return chain.proceed(request)
            }
        })
    }

    override fun getArticles(): LiveData<List<Article>> {
        val _data = MutableLiveData<List<Article>> ()

        val articleList = service.list().execute().body()?.articles ?: listOf()

        // TODO Convertir la liste des articles du modèle du web service vers le modèle métier ArcicleItem --> Article

        val articles = articleList.map {

        }

        _data.value = articles
    }

    companion object {
        private const val apiKey = "YOUR_API_KEY"
        private const val apiUrl = "THE_API_URL"
    }

}
```

## Finaliser l'implémentation 
- Créer un repository pour traiter les requêtes venant de la ui 
- Créeer un ViewModel pour traiter la donnée des fragments 


