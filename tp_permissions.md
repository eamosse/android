# Gérer les permissions
Dans ce TP, nous allons expérimenter la getion de la permission d'accès au capteur GPS. 

## Partie 1: Ajouter les permissions dans le Manifest 
Modifier le Manifest pour ajouter la permission ```ACCESS_FINE_LOCATION```

## Partie 2: Mise en place du desigh 
Modifier le layout pour qu'il ressemble à la figure suivante. 
![Ajoutez un voisin](/permissions.png "Nouveau voisin")

## Partie 3: Gestion des permissions
Mettre à jour le texte du bouton en fonction du statut de la permission. 
- Si la permission n'a jamais été demandée, le texte doit afficher "Bienvenue sur notre application, pour avoir votre position cliquer sur le bouton 'Autoriser l'accès'"
- Si la permission a été refusée, le bouton doit afficher "Vous devez autoriser l'accès au GPS pour qu'on puisse récupérer votre position".

### Quand l'utilisateur clique sur le bouton
- Si la permission n'a jamais été demandé --> Demander la permission 

- Si l'utilisateur a déjà refusé la permission : 
1. Si on peut lui expliquer pourquoi on a besoin de la permission --> Afficher une popup pour lui expliquer la permission. 
> Infos sur la popup 
Titre : Autorisation
Message: "Nous avons besoin de votre accord pour afficher votre position GPS"
Button (OK) : Demander la permission 
Button (Annuler): Ne rien faire 

2. Si on ne doit pas expliquer à l'utilisateur pourquoi on a besoin de la permission --> Demander la permission

## Partie 4: Ressources utiles 
https://developer.android.com/training/permissions/requesting#manage-request-code-yourself

- Vérifier Le statut d'une permission 

```Kotlin
ContextCompat.checkSelfPermission(baseContext, Manifest.permission.ACCESS_FINE_LOCATION)
```
> Cette méthode retourne un Int dont les valeurs peuvent être PackageManager.PERMISSION_GRANTED ou PackageManager.PERMISSION_DENIED

- Vérifier si on doit expliquer à l'utilisateur pourquoi on a besoin de de son autorisation

```Kotlin
if(ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.ACCESS_FINE_LOCATION)) {
            // Afficher une popup pour expliquer à l'utilisateur pourquoi on a besoin de son autorisation
        } else {
            // Demander la permission
        }
```

- Demander une permission 
```Kotlin
requestPermissions(baseContext,
                arrayOf(Manifest.permission.ACCESS_FINE_LOCATION),
                MY_CODE)
```

- Récupérer le choix de l'utilisateur 
```Kotlin
override fun onRequestPermissionsResult(requestCode: Int,
        permissions: Array<String>, grantResults: IntArray) {
    when (requestCode) {
        PERMISSION_REQUEST_CODE -> {
            // If request is cancelled, the result arrays are empty.
            if ((grantResults.isNotEmpty() &&
                    grantResults[0] == PackageManager.PERMISSION_GRANTED)) {
                // Permission is granted. Continue the action or workflow
                // in your app.
            } else {
                // Explain to the user that the feature is unavailable because
                // the features requires a permission that the user has denied.
                // At the same time, respect the user's decision. Don't link to
                // system settings in an effort to convince the user to change
                // their decision.
            }
            return
        }

        // Add other 'when' lines to check for other
        // permissions this app might request.
        else -> {
            // Ignore all other requests.
        }
    }
}
```

## Partie 5: Bonus 
Utiliser la version évoluée de la gestion de permission. 

https://developer.android.com/training/permissions/requesting#allow-system-manage-request-code

