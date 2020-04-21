## I- Creation d'un document (Indexation de document):
* L'indexation de document signifie la création de document en l'affectant à un index. Cela peut se faire par un appel API "PUT" ou "POST":
  - Si on utilise l'appel "PUT" pour la création d'un document, il faut absolument préciser l'**_id** de ce document (sinon on aura un message d'erreur demandant soit de préciser l'id, soit de passer par l'appel "POST" si on ne souhaite pas préciser l'id de doc):
  ```javascript
    PUT {name_index}/_doc/{_id_doc}
    {
        "field1": xxx,
        "field2": yyy
         
    }
  ```
  -  Si on utilise l'appel "POST" pour la création d'un document, on peut préciser ou pas l'**_id** de ce document (si on n'indique pas l'id, ES va générer automatiquement un id aléatoire et unique pour ce doc):

    ```javascript
    POST {name_index}/_doc/{_id_doc}
    {
        "field1": xxx,
        "field2": yyy
         
    }
  ```
  ou
    ```javascript
    POST {name_index}/_doc
    {
        "field1": "xxx",
        "field2": 1
         
    }
  ```
- Remarque:  le moment de création de document, on peut toujours mettre à jour le mapping de l'index (ce pourquoi on dit que ES est schemaless):
  - on peut ajouter des nouveaux fields ou meme changer le type d'un field déjà existant.
  - par exemple dans cet exemple ci-dessous, on changer le type de field2 à "text" et on a ajouté un nouveau field (field4) en lui donnant la valeur (3) => ES va comprendre qu'il doit ajouter un nouveau field (field4) au mapping de notre index, et que son type doit etre "integer" (en se basant sur la valeur qu'on a donné à ce nouveau field)
    ```javascript
    POST {name_index}/_doc
    {
        "field1": "xxx",
        "field2": "1",
        "field4" : 3
         
    }
  ```
## II- Read document by id:

- Ca se fait graçe à l'appel "GET":
```javascript
GET {name_index}/_doc/{id_doc}
```
- Par exemple, cet appel, ci-dessous, va récuperer le document avec l'id (2) de l'index de nom (sogeti_skills_manager): 
```javascript
GET sogeti_skills_manager/_doc/2
```

## II- Update de document by id:

- Ca se fait graçe à un appel "POST":
```javascript
POST {name_index}/_update/{id_doc}
{
    "doc":{
        "field1":"new_value"
    }
}
```
- A noter que:
  - l'opération d'update ne modifie/màj que les champs passés dans le cors de la request, et laisse les autres champs intacts.
  - avec chaque opération d'update d'un document, la version de ce document sera incrémentée.

- Par exemple, si on souhaite mettre à jour le doc d'id : **5rk7ZHEBkCzlIo0Xy1pC** de l'index **sogeti_skills_manager**, en corrigant le champs **skill** du "Kubernetes" à "GCP":
  - on va tout d'abord récuperer la version actuelle de ce doc:
    ```javascript
    GET sogeti_skills_manager/_doc/5rk7ZHEBkCzlIo0Xy1pC
    ```
  - on obtient la réponse ci-dessous. On peut constater que la version de ce doc est égale à 1.
  ```
    {
    "_index" : "sogeti_skills_manager",
    "_type" : "_doc",
    "_id" : "5rk7ZHEBkCzlIo0Xy1pC",
    "_version" : 1,
    "_seq_no" : 7,
    "_primary_term" : 1,
    "found" : true,
    "_source" : {
        "type" : "skill",
        "skills_group" : "Devops",
        "skill" : "Kubernetes",
        "keywords" : [
        "GCP"
        ]
    }
    }
  ```
  on peut constater que la version de ce doc est égale à 1.

  - Maintenant, on va mettre à jour le champs **skill** de ce doc:

    ```javascript
    POST sogeti_skills_manager/_update/5rk7ZHEBkCzlIo0Xy1pC
    {
        "skill" : "GCP"
    }
    ```
  - Si on refait le GET de meme document, on peut constater justement que uniquement ce field **skill** qu'a été modifié (les autres champs restent intacts), et que la version de ce doc a augmenté du 1 à 2:
    ```
    {
    "_index" : "sogeti_skills_manager",
    "_type" : "_doc",
    "_id" : "5rk7ZHEBkCzlIo0Xy1pC",
    "_version" : 2,
    "_seq_no" : 7,
    "_primary_term" : 1,
    "found" : true,
    "_source" : {
        "type" : "skill",
        "skills_group" : "Devops",
        "skill" : "GCP",
        "keywords" : [
        "GCP"
        ]
    }
    }
    ```

## III- Delete Document by Id:
- Ca se fait grace à l'appel "DELETE":
```javascript
DELETE {name_index}/_doc/{id_doc}
```
- Remarque: 
  - Après cet appel "DELETE", **Apache Lucene** ne supprime pas directement ce document. Il le marque uniquement comme "**deleted**". 
  - Ensuite (après un certain intervalle qu'on peut le regler dans le settings), **Lucene** passe à travers les shards pour les nettoyer en supprimant définitivement les docs qui sont marqués comme "**deleted**".
  - Ce pourquoi, lorsque on essaye de recréer un doc **avec le meme _id que le doc qu'a été supprimé** (afin de remplacer le doc qu'a été supprimé), on va remarqué que la version de nouveau document ne sera pas : 1, mais plutot la version incrementée du doc, qu'a été supprimé, portant précedement ce meme _id. (voir l'exemple ci-dessous)

- Exemple: si on souhaite supprimé le document portant l'_id **5rk7ZHEBkCzlIo0Xy1pC** de l'index **sogeti_skills_manager**:
    - on lance l'appel "DELETE":
    ```javascript
    DELETE sogeti_skills_manager/_doc/5rk7ZHEBkCzlIo0Xy1pC
    ```
    - on aura cette réponse, en résultat:
    ```
    {
    "_index": "sogeti_skills_manager",
    "_type": "_doc",
    "_id": "5rk7ZHEBkCzlIo0Xy1pC",
    "_version": 2,
    "result": "deleted",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 14,
    "_primary_term": 4
    }
    ```
    On peut remarquer que dans le champs "**result**" de cette réponse: on a "**deleted**". Cela signifie que Lucene a marqué ce doc comme ""**deleted**". On peut distinguer aussi que la version a été incrementée à 2.
    - Si on essaye de re-récuperer ce meme document par son id, on aura la réponse **not found**:
    ```
    {
    "_index": "sogeti_skills_manager",
    "_type": "_doc",
    "_id": "5rk7ZHEBkCzlIo0Xy1pC",
    "found": false
    }
    ```
    - Si on essaye de re-créer le meme document (avec les memes champs/valeurs) avec le meme _id:
      - le request est:
    ```javascript
    POST sogeti_skills_manager/_doc/5rk7ZHEBkCzlIo0Xy1pC
    {
        "type": "skill",
        "skills_group": "Devops",
        "skill": "Kubernetes",
        "keywords": [
            "GCP"
        ]
    }
    ```
    - Si on essaye de re-récuperer ce doc avec son id:
      - la réponse montrera que la version de ce doc a été incrémenté une nouvelle fois (à 3), et que le champs **result** est marqué **created**:
    ``` 
    {
        "_index": "sogeti_skills_manager",
        "_type": "_doc",
        "_id": "5rk7ZHEBkCzlIo0Xy1pC",
        "_version": 3,
        "result": "created",
        "_shards": {
            "total": 2,
            "successful": 1,
            "failed": 0
        },
        "_seq_no": 17,
        "_primary_term": 4
    }
    ```

