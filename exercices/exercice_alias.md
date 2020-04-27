## Exercice 3:
Tout les appels API / requetes définis dans cet exercice, sont à exécuter sur [Kibana](http://localhost:5601/app/kibana#/dev_tools/console) (sur la console de dev-tools de Kibana).

- **Question 1: Crée trois index et fais les référencié par un alias**
 - On commence par crée trois index:
```javascript
PUT index-1
PUT index-2
PUT index-3
```

 - Après, on va populer chaque index de ces trois, par deux documents chacun:
 ```javascript
    POST index-1/_doc
    {
        "country": "FR"
    }
    POST index-1/_doc
    {
        "country": "ES"
    }
    
    POST index-2/_doc
    {
        "country": "TN"
    }
    POST index-2/_doc
    {
        "country": "ALG"
    }  

    POST index-3/_doc
    {
        "country": "JAP"
    }
    POST index-3/_doc
    {
        "country": "IN"
    }
 ```
 - Après, on va faire réf à ces trois index par le meme alias, qu'on va l'appeler **alias-countries**: (ce nouveau alias va faire réf aux trois index et contenir la somme de leurs docs, çàd les 6 docs):
   - 1ère méthode: Faire des appels séparés
    ```javascript
    POST index-1/_alias/alias-countries
    POST index-2/_alias/alias-countries
    POST index-3/_alias/alias-countries
    ```
    - 2ème méthode: Faire un seul appel
    ```javascript
    POST index-*/_alias/alias-countries
    ```

- **Question 2: Lis tout les docs contenus dans un alias**
  - Afin de récupérer tout les documents contenus dans un alias, on lance l'appel suivant:

```javascript
GET alias-countries/_search
```
  - Suite à cet appel ci-dessus, on obtient la réponse suivante, qui nous montre que notre alias contient en totalité six docs (c'est la somme de docs contenus dans les trois index réferenciés par cet alias):
```
{
  "took" : 20,
  "timed_out" : false,
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 6,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "index-1",
        "_type" : "_doc",
        "_id" : "ACbPt3EBz5esiilLlzdF",
        "_score" : 1.0,
        "_source" : {
          "country" : "FR"
        }
      },
      {
        "_index" : "index-1",
        "_type" : "_doc",
        "_id" : "ASbPt3EBz5esiilLlzec",
        "_score" : 1.0,
        "_source" : {
          "country" : "ES"
        }
      },
      {
        "_index" : "index-2",
        "_type" : "_doc",
        "_id" : "AibPt3EBz5esiilLlze0",
        "_score" : 1.0,
        "_source" : {
          "country" : "TN"
        }
      },
      {
        "_index" : "index-2",
        "_type" : "_doc",
        "_id" : "AybPt3EBz5esiilLmDcI",
        "_score" : 1.0,
        "_source" : {
          "country" : "ALG"
        }
      },
      {
        "_index" : "index-3",
        "_type" : "_doc",
        "_id" : "BCbPt3EBz5esiilLmDce",
        "_score" : 1.0,
        "_source" : {
          "country" : "JAP"
        }
      },
      {
        "_index" : "index-3",
        "_type" : "_doc",
        "_id" : "BSbPt3EBz5esiilLmDd6",
        "_score" : 1.0,
        "_source" : {
          "country" : "IN"
        }
      }
    ]
  }
}
```
- **Question 3: Dé-référencier l'index-1 de notre alias et vérifier combien de docs restent dans cet alias**
  - Au début, l'alias "**alias-countries**" contient 6 docs (comme on a vu dans la question 2).
  - Maintenant, on va supprimer/enlever l'index-1 de cet alias, avec cet appel:
  ```javascript
    DELETE index-1/_alias/alias-countries
  ```

  - Après cet appel, l'alias "**alias-countries**"  ne fait plus réf à l'index-1, donc cet alias ne contient plus les docs de l'index-1. On peut confirmer ça, en re-récupérant tout les docs contenus dans cet alias. On vera que cet alias ne contient que 4 docs (les docs de l'index-1 ne sont plus contenus dans cet alias).