# Gérer la persistance avec Room 
Dans ce TP, nous allons améliorer l'application Entre Voisin développée dans les TPs 2 et 3. 

Jusqu'ici les données de l'application étaient gérées en mémoire, par conséquent quand on redémarre l'application les données sont réinitialisées. 

Nous allons utiliser une base de données pour gérer la persistance afin que les données persistent au remarrage de l'application. 

## Partie 0: Créer une nouvelle branche dans le TP2/3
Ajouter une branche room dans le reposiroty du TP 2/3.

## Partie 1: Mise en place de l'architecture du code 
Le schéma suivant représente l'architecture du code de l'applization 
![Ajoutez un voisin](/architecture.png "Nouveau voisin")

Nous allons réorganiser le code de l'application pour qu'il réflète mieux l'architecture du projet. 

Sous le package principale de l'application : 
- Créer un package ```ui``` puis déplacer l'activity ```MainActivity``` et le package ```fragments``` dans ce nouveau package; 

- Créer un package ```dal```, puis dans ce nouveau package ajouter deux sous-packages ```room``` et ```memory```. 

- Déplacer la classe ```InMemoryNeighborDataSource``` dans le package ```memory```

- Déplacer la classe ```NeighborDatasource``` dans le package ```dal```

- Renommer le package ```data``` en ```repositories``` 

A ce stade, les packages du projet devraient être organisés comme sur la figure suivante : 

![Ajoutez un voisin](/packages.png "Nouveau voisin")

> Assurez-vous que le projet compile

## Partie 2: Mise en place de la Room 

### 1. Dépendances 
Modifier le fichier ```build.gradle``` du module de l'application.

```gradle
  ....
  apply plugin: 'kotlin-kapt'
  ....

  def room_version = "2.2.6"

  implementation "androidx.room:room-runtime:$room_version"
  kapt "androidx.room:room-compiler:$room_version"

```
> Resync le projet, gradle téléchargera automatiquement les dépendances

### 2. Ajouter une nouvelle entité

- Dans le package ```room```, créer un nouveau package ```entities```. Ajouter une classe NeighborEntity dans le package ```entities```

```Kotlin
@Entity(tableName = "neighbors")
class NeighborEntity(
    @PrimaryKey(autoGenerate = true)
    val id: Long,
    var name: String,
    var avatarUrl: String,
    val address: String,
    var phoneNumber: String,
    var aboutMe: String,
    var favorite: Boolean = false,
    var webSite: String? = null,
)
```
> Observez bien le les variables: 
- id est un val et c'est l'a clef primaire
- address est un val car un ne veux pas qu'on puisse changer l'adresse d'un voisin; c'est bête mais vous comprenez l'idée
- website peut être null, tous les voisins ne sont pas des geecks


### 3. Ajouter une DAO 
- Dans le package ```room```, créer un sous-package ```daos```. Ajouter une interface ```NeighborDao``` dans le package ```daos```.
> Be careful, quand vous définissez le package ```daos```, assurez-vous de ne pas l'ajouter dans ```entities```. 

- Modifiez l'interface pour y ajouter une méthode pour récupérer la liste des neighbors dans la base de données 

```Kotlin
@Dao
interface NeighborDao {
    @Query("SELECT * from neighbors")
    fun getNeighbors(): LiveData<List<NeighborEntity>>
}
```

### 4. Ajouter une source de données pour Room

- Dans le package ```room```, ajouter une classe ```NeighborDataBase``` permettant de modéliser la base de données. 

```Kotlin
@Database(
    entities = [NeighborEntity::class],
    version = 1
)
abstract class NeighborDataBase : RoomDatabase() {
    abstract fun neighborDao(): NeighborDao

    companion object {
        private var instance: NeighborDataBase? = null
        fun getDataBase(application: Application): NeighborDataBase {
            if (instance == null) {
                instance = Room.databaseBuilder(
                    application.applicationContext,
                    NeighborDataBase::class.java,
                    "neighbor_database.db"
                )
                    .fallbackToDestructiveMigration()
                    .build()
            }
            return instance!!
        }
    }
}

```

> Pour mieux comprendre : 
- L'annotation ```@Database``` permet de définir les entités et la version de la base de données. Une base de données relationnelle peut contenir plusieurs tables et par conséquent la classe Kotlin modélisant la base de données permet de définir plusieurs entités; une entité = une table dans la base de données. Le numéro de version est très important car il permet d'indiquer à Room quand le schéma de la BDD change. Ainsi, à chaque modification du modèle, il faut incrémenter le numéro de version. 

- La classe qui modélise la base de données doit être abstraite, c'est room qui générera l'implémentation. 

- La base de données doit définir autant de méthodes que de DAO

- Finalement, la base de données doit être un singleton

- fallbackToDestructiveMigration() indique à Room de supprimer la base de données et la recréer quand le modèle de données change. 


### 5. Ajouter une DataSource 
Dans cette section, nous allons définir une classe dont le seul but est de récupérer les données dans la BDD en fonction du protocole défini par l'appliction ```NeighborDatasource```

- Modifier l'interface ```NeighborDatasource``` pour qu'elle retourne un LiveData au lieu d'une liste

```Kotlin
interface NeighborDatasource {
    /**
     * Get all my Neighbors
     * @return [List]
     */
    val neighbours: LiveData<List<Neighbor>>

    /**
     * Deletes a neighbor
     * @param neighbor : Neighbor
     */
    fun deleteNeighbour(neighbor: Neighbor)

    /**
     * Create a neighbour
     * @param neighbor: Neighbor
     */
    fun createNeighbour(neighbor: Neighbor)

    /**
     * Update "Favorite status" of an existing Neighbour"
     * @param neighbor: Neighbor
     */
    fun updateFavoriteStatus(neighbor: Neighbor)

    /**
     * Update modified fields of an existing Neighbour"
     * @param neighbor: Neighbor
     */
    fun updateNeighbour(neighbor: Neighbor)
}
```

- Corriger les erreurs dans la classe ```InMemoryNeighborDataSource``` 

> Créer un fichier ```Data.kt``` dans le package dal et déplacer la variable InMemory_NeighborS dans ce fichier. N'oubliez pas de la rendre publique. 

```Kotlin
private val _neighbors = MutableLiveData<List<Neighbor>>()

    init {
        _neighbors.value = InMemory_NeighborS
    }

    override val neighbours: LiveData<List<Neighbor>>
        get() = _neighbors
......

```
> Analysons ce qui se passe ici 
- Il n'est pas possible de modifier les données (value) d'un LiveData, c'est une instance immuable. Si on veut retourner la liste ```InMemory_NeighborS``` dans un live data on doit utiliser une implémentation qui autorise les modifications, un ```MutableLiveData```. 

Pour cela, nous avons caché l'instance du MutableLiveData pour qu'on ne puisse pas y accéder depuis l'extérieur. Finalement la datasource retourne une version non modifiable du live data.

Cette mécanique est une bonne pratique du clean architecture, on expose toujours des données non modifiables aux autres couches; cela permet de garantir l'intégrité des données. 

- Fixer les erreurs dans le repository 
[A vous de jouer]

- Fixer les erreurs dans le fragment ```ListNeighborsFragment```

```Kotlin
private fun setData() {
        val neighbors = NeighborRepository.getInstance().getNeighbours()
        neighbors.observe(viewLifecycleOwner) {
            val adapter = ListNeighborsAdapter(it, this)
            binding.neighborsList.adapter = adapter
        }
    }
```
> Le repository ne renvoi plus une liste mais un LiveData. Il faut observer le LiveData et mettre la liste à jour à chaque modification de données.


> A ce stade, votre projet doit compiler et fonctionner normalement 


- Dans le package ```room```, ajouter une classe ```RoomNeighborDataSource``` qui implémente l'interface ```NeighborDatasource```

```Kotlin

class RoomNeighborDataSource(application: Application) : NeighborDatasource {
    private val database: NeighborDataBase = NeighborDataBase.getDataBase(application)
    private val dao: NeighborDao = database.neighborDao()

    private val _neighors = MediatorLiveData<List<Neighbor>>()

    init {
        _neighors.addSource(dao.getNeighbors()) { entities ->
            _neighors.value = entities.map { entity ->
                entity.toNeighbor()
            }
        }
    }

    override val neighbours: LiveData<List<Neighbor>>
        get() = _neighors

    override fun deleteNeighbour(neighbor: Neighbor) {
        TODO("Not yet implemented")
    }

    override fun createNeighbour(neighbor: Neighbor) {
        TODO("Not yet implemented")
    }

    override fun updateFavoriteStatus(neighbor: Neighbor) {
        TODO("Not yet implemented")
    }

    override fun updateNeighbour(neighbor: Neighbor) {
        TODO("Not yet implemented")
    }
}

```
- Dans le package ```dal```, ajouter un nouveau package ```utilis``` et dans ce nouveau package ajouter un fichier Kotlin ```NeighborMapper.kt```. 

- Créer une fonction d'extension dans ```NeighborMapper.kt``` permettant de convertir une instance de ```NeighborEntity``` en ```Neighbor```
> Les fonctions d'extension permettent d'implémenter une fonction dans une classe sans la surcharger. 

```Kotlin
fun NeighborEntity.toNeighbor() = Neighbor(
    id = id,
    name = name,
    avatarUrl = avatarUrl,
    address = address,
    phoneNumber = phoneNumber,
    aboutMe = aboutMe,
    favorite = favorite,
    webSite = webSite ?: ""
)
```

> Petite analyse 

- Cette classe implémente ```NeighborDatasource``` qui lui force à avoir le même comportement que la classe ```InMemoryNeighborDataSource```. Ainsi, la seule différence entre les deux sources de données est le mode de persistance. 

- Ici on a utilisé un ```MediatorLiveData``` qui nous permet d'observer les changements sur la base de données et appliquer des modifications sur les données renvoyées par la BDD avant de les afficher à l'utilisateur. 

Cette étape est importante car la base de données retourne une liste de ```NeighborEntity``` alors que la vue attend une liste de ```Neighbor```. 

Là encore on est face à un principe de clean architecture. La base de données à son modèle données qui est différente de celle de la vue. 


Pour faciliter la communication entre la couche de données et la vue, il faut créer une routine permettant de convertir les données au bon format attendu par la couche de destination. 

Ici, on veut renvoyer des données de la base de données à la vue, on a besoin d'une méthode qui convertit une entité en modème métier. Pour cela l'instance du ```MediatorLiveData``` observe les changements sur les données de la BDD et quand elles changent on applique une conversion sur les données avant de mettre à jour le live data. 

Finalement, on convertit le ```MediatorLiveData``` en ```LiveData``` avant de le retourner à la vue. 


### 6. Adapter le repository 
La data source ```RoomNeighborDataSource``` a besoin d'une instance de l'application pour s'initialiser; en fait ce'est pas la base de données qui en a besoin. 

Pour cela, nous allons modifier le repository pour qu'elle accepte en paramètre une instance d'application. 

### 1. Modifier le repository

```Kotlin
class NeighborRepository private constructor(application: Application) {
    private val dataSource: NeighborDatasource

    init {
        dataSource = RoomNeighborDataSource(application)
    }

    // Méthode qui retourne la liste des voisins
    fun getNeighbours(): LiveData<List<Neighbor>> = dataSource.neighbours

    fun delete(neighbor: Neighbor) = dataSource.deleteNeighbour(neighbor)

    companion object {
        private var instance: NeighborRepository? = null

        // On crée un méthode qui retourne l'instance courante du repository si elle existe ou en crée une nouvelle sinon
        fun getInstance(application: Application): NeighborRepository {
            if (instance == null) {
                instance = NeighborRepository(application)
            }
            return instance!!
        }
    }
}
```

> On essaie de comprendre ? 

- On veut maitriser l'instanciation du reposiroty, on met son constructeur en private

- On veut que le reposirory soit un singleton, on crée une méthode statique pour l'initialiser 

- Finalement, on change de source de données; on remplace ```InMemoryNeighborDataSource``` par ```RoomNeighborDataSource``` 

### 2. Fixer les erreurs de compilation
L'instanciation du repository nécessite une instance de application en paramètre.

- Modifier la méthode setData pour récupérer une instance de l'application et la passer en paramètre au repository. 
```Kotlin
private fun setData() {
        // Récupérer l'instance de l'application, si elle est null arrêter l'exécution de la méthode

        val application: Application = activity?.application ?: return
        val neighbors = NeighborRepository.getInstance(application).getNeighbours()
        neighbors.observe(viewLifecycleOwner) {
            val adapter = ListNeighborsAdapter(it, this)
            binding.neighborsList.adapter = adapter
        }
    }
```   

> Essayez de compiler et tester. Que remarquez-vous ? Pourquoi ? 

- Vous devriez avoir une page blanche, ce qui est logique car l'a base de données est vide. 

### Ajouter des données dans la BDD
- Ajouter un callback lors de la l'instanciation de la BDD pour intercepter l'événement de création de la BDD. 
```Kotlin
fun getDataBase(application: Application): NeighborDataBase {
            if (instance == null) {
                instance = Room.databaseBuilder(
                    application.applicationContext,
                    NeighborDataBase::class.java,
                    "neighbor_database.db"
                ).addCallback(object : Callback() {
                    override fun onCreate(db: SupportSQLiteDatabase) {
                        super.onCreate(db)
                        insertFakeData()
                    }
                })
                    .fallbackToDestructiveMigration()
                    .build()
            }
            return instance!!
        }

        private fun insertFakeData() {
            Executors.newSingleThreadExecutor().execute {
                InMemory_NeighborS.forEach {
                    instance?.neighborDao()?.add(it.toEntity())
                }
            }
        }
```
> Nous avons ajouté un callback lors de l'instanciation de la base de données pour intercepté la création de la base de données. 

> Compilez et testez

## Partie 3: Injection de dépendance
Notre repository fonctionne mais le processus permettant de récupérer son instance est un peu lourd; on va l'améliorer en utilisant une injection de dépendance fait maison. 
> On utlisera une librairie de gestion de dépendance dans le prochain TP. 

- Sous le package principal du projet, créer un nouveau package ```di``` et dans ce package ajouter une classe ```DI.kt```
```Kotlin
object DI {
    lateinit var repository: NeighborRepository
    fun inject(application: Application) {
        repository = NeighborRepository.getInstance(application)
    }
}
```

- Modifier la méthode onCreate de l'activity ```MainActivity``` pour instancier le repository once for all :) 
```Kotlin
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        DI.inject(application)
        ...
    }
```

> Pour récupérer l'instance du ViewModel il suffit d'appeler ```DI.repository``` 

- Remplacer les références à ```Repository.getInstance()``` par ```DI.repository``` 


## Partie 4: Utiliser un View Model 
Notre architecture n'est pas totalement clean, car le fragment appel directement le repository. On va utiliser un View Model pour gérer les données du fragment et la communication avec le repository. 

- Ajouter les dépencances aux librairies viewModel et liveData
```gradle
def lifecycle_version = "2.3.0"
implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
implementation "androidx.lifecycle:lifecycle-livedata-ktx:$lifecycle_version"
```

- Sous le package principal du projet, ajouter un package ```viewmodels``` et une nouvelle classe ```RepositoryViewModel``` dans le package ```viewmodels```. 

- Modifier la classe ```RepositoryViewModel``` pour qu'elle implémente ViewModel. 

```Kotlin
class NeighborViewModel : ViewModel() {
    private val repository: NeighborRepository = DI.repository

    // On fait passe plat sur le résultat retourné par le repository
    val neighbors: LiveData<List<Neighbor>>
        get() = repository.getNeighbours()
}
```

- Modifier le fragment ```ListNeighborsFragment``` pour récupérer la liste des voisins via le viewmodel ```NeighborViewModel```. 

1. Instancier le viewModle 
```Kotlin
***
    private lateinit var viewModel: NeighborViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        viewModel = ViewModelProvider(this).get(NeighborViewModel::class.java)
    }
***
```
> Avez-vous remarqué ? 
- On ne créer pas une instance de viewModel mais on la récupére dans un provider ```ViewModelProvider```. L'appel à la méthode ```get``` vérifie si une instance du viewmodel existe, si oui il le retourne; sinon il crée une nouvelle instance, l'ajouter dans le store et le retourner.

- Un ```ViewModelProvider``` est lié soit à une activité ou un fragment. Ici, ```ViewModelProvider(this)``` fait référence au gestionnaire de viewmodel du fragment. Ainsi, tant que le owner du viewmodel existe, le viewmodel existe aussi, quand le owner est détruit; il est détruit aussi.

- Si on veut partager un viewmodel entre deux fragments, il faut le le lier à l'activité et non à chaque fragment. 

2. Observer le livedata exposant la liste des voisins dans le fragment

```Kotlin
private fun setData() {
        viewModel.neighbors.observe(viewLifecycleOwner) {
            val adapter = ListNeighborsAdapter(it, this)
            binding.neighborsList.adapter = adapter
        }
    }

``` 

## Partie 5: A vous de jouer
Maintenant vous allez coder les amis :) 

1. Ajouter les méthodes pour supprimer un voisin quand on clique sur l'icone suppression. 

> ATTENTION, vous n'avez pas besoin de raffraichir la liste, elle sera mise à jour automatiquement via le live Data

2. Ajouter les méthodes pour ajouter un nouveau voisin dans la base de données à partir du formulaire créé dans les TP 2 et 3. 

3. Ajouter un menu dans la toolbar (en haut à droite) permettant à l'utilisateur de choisir la source de données à utiliser : 
- En mémoire
- Dans une base de données

4. Ajouter une vue pour afficher le détail d'un voisin. 

5. Ajouter un bouton dans la vue de détail permettant d'ajouter un voisi en favori. 
