## Exercice 2:
Tout les appels API / requetes définis dans cet exercice, sont à exécuter sur [Kibana](http://localhost:5601/app/kibana#/dev_tools/console) (sur la console de dev-tools de Kibana).

- **Question n°1: Créer un index nommé **names**: avec deux fields: 
  - **name* : dont le type est **keyword* (ne supporte pas le **full text search*)
  - et **name_text* : dont le type est **text* (supporte bien le **full text search* [voir le remarque 2, ci-dessous])

```javascript
PUT names
{
  "mappings": {
    "properties": {
      "name":{
        "type": "keyword"
      },
      "name-text":{
        "type": "text"
      }
    }
  }
}
```
- Remarques:
  - Remarque 1: Après lancer l'appel "PUT" ci-dessus, ES crée un index **names** avec le mappings précisé dans le body request. Mais voir qu'on n'a pas précisé le settings de cet index, ES va mettre de settings par défaut (number_of_replicas=1, number_of_shards=1).
  Faisons un appel "GET" de cet index, et nous verrons que ce setting par défaut est vrmt mis en place:

```javascript
  {
  "names" : {
    "aliases" : { },
    "mappings" : {
      "properties" : {
        "name" : {
          "type" : "keyword"
        },
        "name-text" : {
          "type" : "text"
        }
      }
    },
    "settings" : {
      "index" : {
        "creation_date" : "1586852743728",
        "number_of_shards" : "1",
        "number_of_replicas" : "1",
        "uuid" : "9TWHmftWSXGqPVMxPCACzA",
        "version" : {
          "created" : "7060299"
        },
        "provided_name" : "names"
      }
    }
  }
}
```
  - Remarque 2: Contrairement aux fields de type _keyword_, un field de type _text_ autorise l'option de **full text search**.
  Cette option nous permet de chercher un doc en se basant sur uniquement une partie de texte de field en question. Par contre, si on souhaite faire la recherche en se basant sur un field de type **keyword**, on devra écrire tout le texte afin de tomber sur le bon doc, sinon on aura **document introuvable**. 
  Par exemple, si on crée un document avec les fields: **name**= "Heithem Lejmi" et **name_text**= "Heithem Lejmi"
  Si on veut chercher ce document, en écrivant dans la query de search: "name" = "Heithem", on ne va pas trouver le document, car ce field **name** est de type keyword, donc on doit écrire tout le texte "Heithem Lejmi" pour trouver ce doc.
  Mais si on faire le search en mettant dans le query: "name_text"="Heithem", on va bien trouver le document, car ce field "name_text" est de type texte et donc il autorise l'option de **full text search**

- **Question n°2: Crééer des docs dans cet index "names":**

  - 1ère mèthode: On crée deux documents dans cet index, avec deux appels "PUT" séparés 
```javascript
PUT names/_doc/1
{
  "name": "Heithem Lejmi",
  "name-text": "Heithem Lejmi"
}

PUT names/_doc/2
{
  "name": "Cristiano Ronaldo",
  "name-text": "Cristiano Ronaldo"
}
```

  - 2ème méthode: on crée les deux documents dans un seul appel avec l'API **BULK**:

```javascript
POST names/_doc/_bulk
{"index":{}}
{"name":"Heithem Lejmi","name_text":"Heithem Lejmi"}
{"index":{}}
{"name":"Cristiano Ronaldo","name_text":"Cristiano Ronaldo"}

```

- **Question n°3: Récuperer tous les docs dans cet index "names":**
 - 1ère méthode: Avec un appel "GET" et sans un body request
 ```javascript
 GET names/_search
```

 - 2ème méthode:Avec un appel "POST" et avec un body request contenant une query de search avec un **match_all** pour tout récupérer dans cet index:
```javascript
POST names/_search
{
  "query": {
    "match_all": {}
  }
}
```

- **Question n°4: Faire une recherche d'un doc dans cet index "names":**
  - Ici, on va découvrir la différence entre faire un search avec un field de type _keyword_ et un autre de type _text_:
  - **1er cas : search avec un field de type keyword** : Si par exemple, on va faire une recherche du document contenant le name "Heithem", sachant que name est de type **keyword** et qu'on a un doc contenant déjà le field name = "Heithem Lejmi":
    - L'appel de search dans ce cas sera:
  ```javascript
  POST names/_search
  {
    "query": {
      "match": {
        "name": "Heithem"
      }
    }
  }
  ```
    - Cet appel donne not found, car on ne peut pas faire "full text search" pour un champs de type keyword:
  ```
  {
    "took" : 6,
    "timed_out" : false,
    "_shards" : {
      "total" : 1,
      "successful" : 1,
      "skipped" : 0,
      "failed" : 0
    },
    "hits" : {
      "total" : {
        "value" : 0,
        "relation" : "eq"
      },
      "max_score" : null,
      "hits" : [ ]
    }
  }
  ```
    - Pour avoir un résultat quand on fait de search avec un field de type **keyword**, il faut entrer tout le text (pas une partie) et aussi de la meme manière (çàd en respectant la manière avec laquelle on a entré ce texte: par exp il faut respecter les majuscule/miniscule):
  ```javascript
    POST names/_search
    {
      "query": {
        "match": {
          "name": "Heithem Lejmi"
        }
      }
    }
  ```
    - Cet appel donne le bon document, car on a passé exactement le meme texte "Heithem Lejmi" au field "name" (de type keyword):
  ```
  {
    "took" : 0,
    "timed_out" : false,
    "_shards" : {
      "total" : 1,
      "successful" : 1,
      "skipped" : 0,
      "failed" : 0
    },
    "hits" : {
      "total" : {
        "value" : 1,
        "relation" : "eq"
      },
      "max_score" : 0.6931471,
      "hits" : [
        {
          "_index" : "names",
          "_type" : "_doc",
          "_id" : "1",
          "_score" : 0.6931471,
          "_source" : {
            "name" : "Heithem Lejmi",
            "name-text" : "Heithem Lejmi"
          }
        }
      ]
    }
  }
  ```  
  - **2ème cas : search avec un field de type text** en faisant la recherche: avec un paramètre de type "text", nous permet de faire de "full text search". C'est à dire, on peut chercher le doc, avec une partie de texte, et sans besoin d'écrire ce texte exactement comme on l'est entré la 1ère fois (on est pas obligé de respecter les maj/min):
    - L'appel, en écrivant seulement une partie de texte et sans respecter les majuscules (on a écrit "heithem", alors que le field "name_text" est défini comme "Heithem Lejmi")
  ```javascript
  POST names/_search
  {
    "query": {
      "match": {
        "name-text": "heithem"
      }
    }
  }
  ```
    - Le résulat de recherche est bien le doc qu'on cherche, malgré que dans ce doc: "name_text" = "Heithem Lejmi", et quion a passé dans notre query de search un argument "heithem":
  ```
  {
    "took" : 15,
    "timed_out" : false,
    "_shards" : {
      "total" : 1,
      "successful" : 1,
      "skipped" : 0,
      "failed" : 0
    },
    "hits" : {
      "total" : {
        "value" : 1,
        "relation" : "eq"
      },
      "max_score" : 0.6931471,
      "hits" : [
        {
          "_index" : "names",
          "_type" : "_doc",
          "_id" : "1",
          "_score" : 0.6931471,
          "_source" : {
            "name" : "Heithem Lejmi",
            "name-text" : "Heithem Lejmi"
          }
        }
      ]
    }
  }
  ```
- **Question n°5: Modifie le doc d'_id = 2, en changeant "Cristiano Ronaldo" par "CR7":**
  - On commence par récupérer ce doc, pour vérifier s'il existe ou pas, et pour noter sa version (avant la modification):
  ```javascript
  GET names/_doc/2
  ```
  - Cet appel "GET" nous donne le bon doc, et on peut remarquer que la version est = 1:
  ```javascript
  {
  "_index" : "names",
  "_type" : "_doc",
  "_id" : "2",
  "_version" : 1,
  "_seq_no" : 19,
  "_primary_term" : 5,
  "found" : true,
  "_source" : {
    "name" : "Cristiano Ronaldo",
    "name-text" : "Cristiano Ronaldo"
    }
  }
  ```
  - Maintenant, on va updater ce doc, en chageant les deux fields: _name_ et **name_text*:
  ```javascript
   POST names/_update/2
  {
    "doc": {
      "name": "CR7",
      "name_text": "CR7"
    }
  }
  ```
  - Suite à cet appel, les deux fields mentionnés dans le body request ont été modifié, la valeur de version est incrémenté et le doc porte le label updated:
  ```
  {
  "_index" : "names",
  "_type" : "_doc",
  "_id" : "2",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
    },
  "_seq_no" : 20,
  "_primary_term" : 5
  }
  ```
  - Si on re-récupère ce mm doc après l'update, on obtiendra:
  ```
  {
  "_index" : "names",
  "_type" : "_doc",
  "_id" : "2",
  "_version" : 2,
  "_seq_no" : 20,
  "_primary_term" : 5,
  "found" : true,
  "_source" : {
    "name" : "CR7", 
    "name-text" : "CR7"
    }
  }
  ```