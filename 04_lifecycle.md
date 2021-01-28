# Cycle de vies des composants 

Dans cet exercie, nous allons expérimenter la gestion du clycle de vies des composants d'une application Android. L'objectif est d'annalyser le comportement d'une activité aux regards des différents événements du système. 

## 1. Créer une application (TP_04_Nom_Prenom)
Créer une nouveau projet en respectant la convention de nommage. 

## 2. Modifier l'application
Modifier le module de l'application en respectant les contraintes suivantes : 

- Modifier le layout de MainActivity pour y ajouter un bouton (Changer d'activité)
- Ajouter une autre activité SecondActivity
- Modifier le layout de la seconde activité pour y ajouter également un bouton (Fermer)

## 3. Traiter les événements
- Traiter les événements de boutons 
    - Quand l'utilisateur clique sur le bouton (Changer d'activité) dans MainActivity, lancer l'activité SecondActivity
    - Quand l'utilisateur clique sur bouton (Fermer) dans SecondActivity, fermer l'activité

## 4. Observez et notez le lifecycle
Modifier les 2 actitivtés pour implémenter dans chacune les méthodes du cycle de vie 
suivantes: 
- onCreate
- onStart
- onResume
- onPause
- onStop
- onRestart
- onDestroy

Ajouter une ligne de logs dans chacune de ces méthodes avec d'observer quand elles sont déclenchées. 

Utiliser la classe Log pour logger les événéments dans la catégories ```debug````
Exemple
```java
Log.d(TAG, "La méthode onCreate est appelée")
```

## 5. Tester l'application 
Compiler et tester l'application en suivant les scénarios suivantes. Il faudra noter à chaque fois les méthodes qui sont appelées dans chaque activité (le séquencement des appels est important).

1. Cliquer sur le bouton (Changer d'activité)
2. Cliquer sur le bouton (Fermer)
3. Appuyer sur le bouton back du téléphone
4. Relancer l'application
5. Demander à un ami de vous appeler
6. Revenir sur l'application après l'appel
7. Cliquer sur le bouton (Changer d'activité)
8. Demander à un ami de vous appeler
9. Revenir sur l'application après l'appel 
10. Vérouiller l'écran du téléphone
11. Dévérouiller l'écran du téléphone 

## 6. Noter les méthodes exécutées
Noter dans un tableau (voir l'example) les méthodes exécutées dans chaque Activity pour chaque action. Si plusieurs méthodes sont exécutées, séparer les par une virgule. Ne rien mettre si aucune méthode n'est exécutée. 

Vous pouvez créer le tableau dans un document Word que vous commiterez avec les sources du projet. 

Pour ceux qui connaissant Markdown, vous pouvez ajouter le tableau dans le readme du projet.

Voici l'example de code que j'ai utilisé pour créer le tableau ci-après.

```markdown
| #Action       | MainActivity      | SecondActivity |
| ------------- |:-----------------:| --------------:|
| 1             | onCreaon, Destroy     | onCreate   |
| 2             | Destroy     | onPause   |

```


| #Action       | MainActivity      | SecondActivity |
| ------------- |:-----------------:| --------------:|
| 1             | onCreaon, Destroy     | onCreate   |
| 2             | Destroy     | onPause   |

## 7. Modalité de rendu
Commitez les sources de votre projet sur Github, Gitlab ou Bitbucket et ajoutez le lien dans le tableau 4 du fichier drive partagé. 
https://docs.google.com/document/d/1h9u-2l4cDpLj28KG4njrYGjATxv4sTGbcVDBAS-1D00/edit



