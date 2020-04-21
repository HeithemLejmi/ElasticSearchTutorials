## Index- Alias
### I- Definition:
- Index-alias est un nom secondaire, utlisé pour faire référence à 1 ou plusieurs index existants.
  - Autrement dit, un seul index-alias peut représenter/faire réf à un ou plusieurs index.

- Index-alias est comme un _virtual item_ utilisé pour représenter un ensemble formé d'1 ou plusieurs vrai items.

### II- Cas d'usage d'Index-Alias:

#### 1- Utiliser l' Alias dans les opérations de "_search":
- Si on souhaite faire un "_search" dans plusieurs index au meme temps:
  - On peut faire plusieurs appels de **_search** dans tous les index:
    ```javascript
    GET index_1/_search
    GET index_2/_search
    GET index_n/_search
    ```
  - Voir que l'index-alias contient tout les docs de tous les index (à lequel il fait réf), donc à la place de faire tous ces appels (un appel par index), on peut faire simplement un **seul appel** de "**_search** dans l'index-alias, qui fait réf à ces plusieurs index:
    ```javascript
    GET {name_alias}/_search
    ```

#### 2- Utiliser l' Alias dans le code de l'application client:
- Problématique: 
  - **a.** Si on mentionne le nom de l'index dans plusieurs endroits de code de l'application (java, python..), et après on souhaite rename cet index (pour une raison **b**), on sera obliger de fouiller le code pour changer le nom de cet index plusieurs fois.
  - **b.** Quand on souhaite renamer un index ??  
    - On sait qu'on ne peut pas modifier le nombre de shards d'un index après sa création. 
    - Pour modifier le nombre de shards d'un index, nous sommes obligés de changer son nom: **On change le nom de l'index et cette fois-ci on doit modifier le nombre de shards**.
    - L'inconvénient de cette  technique: si on utilise le nom de cet index dans plusieurs endroits de code source (java,..) de l'application web.
- Solution: 
On utilise le nom de l'alias d'index (dans le code) à la place d'utiliser directement le nom de l'index (pour faire réf à cet index). 
De cette façon, si par exemple on a un index-alias nommé **index-alias-1** qu'englobe deux index: **index-1** et **index-2**. Si on souhaite renamer l'index **index-1** pour devenir **index-3** (pour augmenter son nombre de shards par exemple), on ne touche plus le code source, il suffit simplement de modifier le settings de l'index-alias **index-alias-1** dans l'ES, afin que cet alias fera référence au nouveau nidex: **index-3** à la place de l'index **index-1**.
- Conclusion: En utilisant l'index-alias dans le code source, on aura plus besoin de modifier ce code, car ce dernier pointe sur l'alias pas sur l'index directement. De cette façon on augmente la qualité et la maintenabilité de notre code source.

### III- Création d'Index-Alias:
#### 1- Création d'Index-Alias sans filtrer les docs:
- Créer un index-alias, revient à dire à ES que cet **index-alias** va faire réf et un tel et tel index.
- Cela se fait grace à un appel "POST":
```javascript
POST {name_of_index}/_alias/{name_alias}
```
- Exemple: Si on a 3 index: **index-1**, **index-2** et **index-3**, dont chaque index contient 8 documents, et qu'on souhaite attribuer tous ces trois index à un seul index-alias nommé **index-alias-1**: on devra faire trois appels "POST" (un appel par index)
```javascript
POST index-1/_alias/index-alias-1
POST index-2/_alias/index-alias-1
POST index-3/_alias/index-alias-1
```
ou dans un seul appel
```javascript
POST index-*/_alias/index-alias-1
```

De cette façon, l'index-alias "**index-alias-1**" va faire réf aux index 1, 2 et 3 et donc va contenir la somme de docs contenus déjà dans ce docs (c'est à dire une totalité de 24 docs).

#### 2- Création d'Index-Alias avec filtrage des docs:
- En utilisant l'option "**filter**" dans notre request au moment de création de l'index-alias, ce dernier sera appliqué que sur les docs retournés par ce filtre:
```javascript
POST index-*/_alias/index-alias-1
{
  "filter":{
    "term":{
      "country": "Tunisia"
    }
  }
}
```
De cette façon, l'index-alias "**index-alias-1**" va contenir tout les docs dont le champs "country" est égal à "Tunisia" dans tout les index dont le nom commence par "index-"  (çàd: les index 1, 2 et 3).

### IV- Faire de _search dans un Index-Alias: 
- Pour récuperer les docs de tous les index représentés par un index-alias:
```javascript
GET {name_of_alias}/_search
```
### V- Delete d'un index de son Index-Alias:
- Delete d'un index de son Index-Alias = Ca revient à dé-réferencier cet index par notre alias.
- Ca se fait grace à un appel "DELETE":
```javascript
DELETE {name_of_index}/_alias/{name_of_alias}
```
- Après cet appel:
  - l'index **{name_of_index}** ne sera plus référencié par l'alias **{name_of_alias}**: çàd que le doc de cet index ne sera plus contenu dans cet alias.
  - mais cet index reste tjrs présent (si on fait **GET** de cet index, il sera **found**)

- Exemple: Si on suppose que l'index-alias "**index-alias-1**" fait déjà réf aux index 1, 2 et 3 et qu'on souhaite effacer l'index 1 "**index-1**" de cet alias, on doit lancer cet appel:
```javascript
DELETE index-1/_alias/index-alias-1
```
  - Après cet appel, l'alias **index-alias-1** ne fait plus réf à l'index **index-1**, pourtant il fait réf qu'à deux index restants : 2 et 3.
  - Si on continue à supprimer les deux index restants de cet alias:
  ```javascript
  DELETE index-*/_alias/index-alias-1
  ```
  Et qu'on essaye de récuperer les docs contenus dans cet alias, avec l'appel **GET** on tombera sur une réponse **not found**, çàd que cet alias n'existe plus (car il ne fait plus réf à aucun index).