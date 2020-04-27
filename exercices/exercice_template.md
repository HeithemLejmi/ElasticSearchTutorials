## Exercice 4:
Tout les appels API / requetes définis dans cet exercice, sont à exécuter sur [Kibana](http://localhost:5601/app/kibana#/dev_tools/console) (sur la console de dev-tools de Kibana).

- **Question 1: Crée un template avec un index_patterns = "elastic-log*"**
  - Afin de créer le template, on lance l'appel suivant, en définissant dans le body request: l'**index_patterns**, les **settings** et les **mappings**:
```javascript
PUT _template/elastic-logs
{
    "index_patterns":["elastic-log-*"],
    "settings":{
        "number_of_shards":2,
        "number_of_replicas":0
    },
    "mappings":{
        "properties":{
            "message":{
                "type":"text"
            },
            "ref":{
                "type":"keyword"
            }
        }
    }
}
```
  - Remarque: l'index_patterns est un array, donc on peut définir plusieurs patterns dans le meme template

- **Question 2: Crée un index en utilisant ce un template**
  - Afin d'appliquer automatiqement le template au moment de création de l'index, il faut que le nom de cet index match l'**index_patterns** de template:
    ```javascript
    PUT elastic-logs-1
    ```
  - Afin de vérifier que cet index récemment créé, suit effectivement notre template, on peut faire un appel "GET" de cet index et comparer les settings & mappings de cet index avec ceux de template:
    - Cet appel de "GET":
    ```javascript
    GET elastic-log-1
    ```
    - Cet appel donne la réponse suivante:
    ```javascript
    {
    "elastic-log-1" : {
        "aliases" : { },
        "mappings" : {
            "properties" : {
                "message" : {
                    "type" : "text"
                    },
                "ref" : {
                    "type" : "keyword"
                    }
                }
            },
        "settings" : {
            "index" : {
                "creation_date" : "1587942087869",
                "number_of_shards" : "2",
                "number_of_replicas" : "0",
                "uuid" : "pkCT7687RUaqDH6Mc6AecA",
                "version" : {
                    "created" : "7060299"
                    },
                "provided_name" : "elastic-log-1"
                }
            }
        }
    }
    ```
    - On peut conclure que l'index "**elatsic-log-1**" et le template "**elastic-logs**" ont les deux, les memes settings & mappings.