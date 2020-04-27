## Exercice 1:
Tout les appels API / requetes définis dans cet exercice, sont à exécuter sur [Kibana](http://localhost:5601/app/kibana#/dev_tools/console) (sur la console de dev-tools de Kibana).

- **Question 1 : Créer un index avec 2 shards et 2 replicas (1 replica by shard)**

```javascript
PUT big-index
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1
  }
}

```
- **Question 2 :  Listez tous les shards (y inclus leur replicas) de l'index "big-index"**
```javascript
GET _cat/shards/big-index
```

- **Question 3 : Lister les index présents sur ce noeud (le serveur):**
```javascript
GET _cat/indices
```
=> On peut remarquer que l'état de cluster, pour l'index "big-index" est **yellow**. Cela est du au fait qu'ES n'arrive pas à assigner les replicas de cet index sur le serveur.
- Au fait, pour l'index "big-index", on a 2 shards et 2 replicas:
 - Voir qu'on lance ES en local **-->** on a un seul serveur 
 - Sur ce seul serveur, ES va déployer les deux shards, mais il ne peut pas déployer les deux replicas aussi sur le meme serveur que leurs shards d'origine (Règle: on lance jamais un replicas sur le meme noeud avec son shard d'origine).
 - Le fait que les replicas ne sont pas encore assignés rend l'état de cluster **yellow**.
 - la solution: 
   - Soit on ajoute un autre serveur, et ES va automatiquement comprendre qu'il y a un nouveau serveur disponible et il va lancer les deux replicas sur ce nouveau serveur (pour les séparer de leurs shards d'origine).
   - Soit on élimine les replicas **=>** de cette façon, on aura un index avec deux shards et avec 0 replicas.

- **Question 4 :  Appliquer la 2ème solution: Mettez à jour les settings de l'index afin d'éliminer le nb de replicas:**

```javascript
PUT big-index/_settings
{
  "number_of_replicas": 0
}
``` 

- **Question 5 : Listez les index présents sur ce noeud et qui commence avec "big*":**
```javascript
GET _cat/indices/big*?v
```

- **Question 6 : Listez les index présents sur ce noeud et qu'ont un état _green_ (pour vérifier que l'état de notre index _big-index_ est devenu green):**
```javascript
GET _cat/indices/?v&health=green
```

- **Question 7 : Une fois qu'on a terminé l'exercice, supprimez cet index:**
```javascript
DELETE big-index 
```

