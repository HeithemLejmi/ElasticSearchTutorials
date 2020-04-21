# Opérations Index

## I- Create Index:
Au moment de creation de l'index, on définit:
- son nom (qu'est son identifiant unique)
- ses settings (nombre de shards et replicas, refresh interval)
- son mappings (ses proprietés (= fields ou column) et leurs types)

```javascript
PUT {name_index}
{
    "settings": {
        "number_of_shards": 2,
        "number_of_replicas":1,
        "refresh_interval":1
    },
    "mappings":{
        "properties":{
            "field1":{
                "type":"text"
            },
            "field2":{
                "type":"integer"
            }
        }
    }
}
```

## II- Update the index's settings & mappings:
Tout les champs de mappings et settings sont modifiables et suceptibles de mise à jour, *** sauf le nombre de shards qu'est non-modifiable ***:

* Mettre à jour, par exemple le nombre de replicas:

```javascript
PUT {name_index}/_settings
{

    "number_of_replicas":1

}
```

* Mettre à jour, par exemple le type d'un field déjà existant (chgt de type de field 1: text -> keyword), ou meme ajouter un nouveau field:

```javascript
PUT {name_index}/_mappings
{

    "properties":{
        "field1":{                
            "type":"keyword"
            },
        "field3":{
            "type":"integer"            
            }
        }

}
```

## III- Lire les settings & mappings d'un index:

Cet appel permet de lire et récupérer les settings & mappings de l'index: 

```javascript
GET {name_index}
```

ou séparemment:

```javascript
GET {name_index}/_settings
GET {name_index}/_mappings
```

## IV- Lister les index présents sur le noeud (serveur):

* Pour lister tout les index:

```javascript
GET _cat/indices
```
* Pour lister tout les index, dont le statut de santé est **green**, ou tout les index dont le nom commence avec "sogeti":

```javascript
GET _cat/indices/?v&health=green

GET _cat/indices/sogeti*?v
```
* Pour lister tout les shards (de tout les index ou d'un index particulier):

```javascript
GET _cat/shards

GET _cat/shards/{name_index}
```