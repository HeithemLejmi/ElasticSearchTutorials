## Exercice 1:
Tout les appels API / requetes définis dans cet exercice, sont à exécuter sur [Kibana](http://localhost:5601/app/kibana#/dev_tools/console) (sur la console de dev-tools de Kibana).

- **Question 1 : Créer un index "old-index" avec deux documents et un seul shard, et re-indexer son contenu dans un nouveau index "new-index" ayant plus de shards**

    - Créer un index-source nommé **old-index** et le populer avec deux docs:
    ```javascript
    PUT old-index
    {
        "settings":{
            "number_of_shards":1,
            "number_of_replicas":0
        },
        "mappings":{
            "properties":{
                "country":{
                    "type":"text"
                }
            }
        }
    }


    PUT old-index/_doc/1
    {
        "country": "FR"
    }
    PUT old-index/_doc/2
    {
        "country": "EN"
    }
    ```
    - Initialiser l'index-dest (initialiser les settings (nb de shards) dans ce cas):
    ```javascript
    PUT new-index
    {
        "settings":{
            "number_of_shards":2,
            "number_of_replicas":0
        }
    }

    ```

    - Re-indexer/Copier le contenu de cet index-source "**old-index**" vers l'index-dest. "**new-index**":

    ```javascript
    POST _reindex
    {
        "source":{
            "index":"old-index"
        },
        "dest":{
            "index":"new-index",
            "version_type": "external"
        }
    }
    ```

    - Si on récupère maintenant le "**new-index**" avec un appel GET, on va constater que la re-indexation a copié uniquement les docs dés l'index source vers l'index-dest sans toucher les settings (la re-indexation ne copie jamais les settings)
        - L'appel GET pour récuperer l'index-dest:
        ```javascript
        GET new-index
        ```
        - La réponse obtenue après cet appel GET, montrant qu'on n'a pas copié les settings de l'index-source, et qu'on a conservé le settings de l'index-dest, et qu'on a copié que les docs:
        ```javascript
        {
        "new-index" : {
            "aliases" : { },
            "mappings" : { },
            "settings" : {
            "index" : {
                "creation_date" : "1587947427873",
                "number_of_shards" : "2",
                "number_of_replicas" : "0",
                "uuid" : "GMEJ-zvZQg2pMezzGIKw4w",
                "version" : {
                "created" : "7060299"
                },
                "provided_name" : "new-index"
            }
            }
        }
        }
        ```

- **Question 2 : Ajoute un nouveau doc dans l'index-source et refais la re-indexation (avec l'option: version_type=external). Qu'est-ce-que vous remarquez ?**
    - Ajouter un nouveau doc:
    ```javascript
    PUT old-index/_doc/3
    {
        "country": "IT"
    }
    ```

    - Refaire la re-indexation avec l'option **version_type**:
    ```javascript
    POST _reindex
    {
        "source":{
            "index":"old-index"
        },
        "dest":{
            "index":"new-index",
            "version_type": "external"
        }
    }
    ```
    
    - Suite à cet appel de re-indexation, on obtient la réponse suivante:
    ```javascript
    {
    "took":13,
    "timed_out":false,
    "total":3,
    "updated":0,
    "created":1,
    "deleted":0,
    "batches":1,
    "version_conflicts":2,
    "noops":0,
    "retries":{
        "bulk":0,
        "search":0
    },
    "throttled_millis":0,
    "requests_per_second":-1.0,
    "throttled_until_millis":0,
    "failures":[
        {
            "index":"new-index",
            "type":"_doc",
            "id":"1",
            "cause":{
                "type":"version_conflict_engine_exception",
                "reason":"[1]: version conflict, current version [1] is higher or equal to the one provided [1]",
                "index_uuid":"0wSo5-3zTeqe9RAoQhiC8Q",
                "shard":"0",
                "index":"new-index"
            },
            "status":409
        },
        {
            "index":"new-index",
            "type":"_doc",
            "id":"2",
            "cause":{
                "type":"version_conflict_engine_exception",
                "reason":"[2]: version conflict, current version [1] is higher or equal to the one provided [1]",
                "index_uuid":"0wSo5-3zTeqe9RAoQhiC8Q",
                "shard":"0",
                "index":"new-index"
            },
            "status":409
        }
    ]
    }
    ```
    - En lisant cette réponse ci-dessus, on peut voir que **version_conflicts** est égale à deux, çàd qu'il y a deux docs qui causent un conflit de version.
    - Je vous rappelle que :
        - l'index-source contient 3 docs
        - la 1ère re-indexation a copié les deux docs d'_id = 1 et 2 dés index-source vers index-dest. Donc ces deux docs ont la version = 1, dans les deux index source et dest.
        - la 2ème re-indexation va copier les trois docs d'id = 1, 2 et 3 dés l'index-source et l'index-dest. Mais, les docs d'id 1 et 2 existent déjà dans l'index-dest.
        - Dans cette situation, ES va regarder l'option "**version_type**", dans ce cas c'est égal à **external**. Cela signifie que ES va vérifier :
          - Si le doc existe déjà (son id est présent dans le doc dest.):
          ES va vérifier la version de ce doc dans l'index-dest, si elle est plus ancienne que celle dans l'index source, ES va mettre à jour ce doc et va préserver la version de l'index source.
          (C'est le cas pour les doc d'_id= 1 et 2, qu'existent déjà dans l'index-dest, mais leur version est = 1 et = à celle dans l'index-source, donc ES ne fait rien).
          - Si le doc n'existe pas encore dans l'index-dest, ES va le créer (c'est le cas pour le doc d'_id = 3)