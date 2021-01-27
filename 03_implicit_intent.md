# Les filtres d'intention

Dans ce TP, nous allons expérimenter les filtres d'intention. L'objectif est de lancer une application à partir d'une autre. 

Pour cela, nous allons créer une application qui comporte une seule activité. L'activité principale contient 4 boutons : 

- Gmail : Quand on clique sur ce bouton, on lance l'application Gmail afin d'envoyer un e-mail avec un texte prédéfini; 

- Appel : Quand on clique sur ce bouton, on lance l'application permettant de placer un appel téléphonique;

- Google: Quand on clique sur ce bouton, on lance Google Chrome pour afficher un lien;

- TP01 : Quand on clique sur ce bouton, on lance l'application développée dans le TP01. 

## Modifier l'application du TP01
Pour permettre le lancement de l'application TP01 depuis une autre application. Pour cela, il faut ajouter un filtre d'intention avec les propriétés suivantes : 
- action : mbds.haiti.TOASt
- category: android.intent.category.DEFAULT

Exemple
```xml
<intent-filter>
    <action android:name="gestion.mbds.CALC" />
    <category android:name="android.intent.category.DEFAULT" />
</intent-filter>
```
