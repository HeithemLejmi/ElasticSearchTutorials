# Operations Index-Template:
## I- Definition & Cas d'usage:
- **1.** Le template d'index définit : les settings + mappings, qu'on peut les re-utiliser et les appliquer automatiquement 
au moment de création de nouveaux index (à la place d'écrire les settings & mappings,  à chaque fois qu'on souhaite créer un index, il suffit maintenant de définir un index-template et de l'appeler au moment de création d'index pour l'utiliser afin de créer cet index).

**=>** Donc comme son nom indique, l'index-template sert commme un blueprint re-utilisable pour la création de autres instances des index qu'auront tous ses meme settings & mappings.

- **2.** ES applique les templates aux nouveaux index à créer, en se basant à un **index_patterns** qui match le nom de l'index à créer.

**=>** L'**index_patterns** est un paramètre qu'on définit au moment de création/défintiion de l'index-template, et qui précise aux quels index ce template s'appliquera.

- **3.** Le changement au niveau de template n'affecte pas les index déjà existants (qu'ont été créé par ce template).

**=>** Ce changement au niveau de template va affecter seulement les nouveaux index qui seront créés par ce template, et pas les index qu'on a déjà créé par ce template.

- **4.** Si on souhaite créer un nouveau index, dont le nom match l'index_patterns de notre template, mais qu'on passe des settings & mappings dans le body de API request de la création de l'index: Quels settings & mappings seront appliqués dans ce cas : ceux de template ou ceux définis dans le body request de l'appel API de create index ?

**=>** Ces sont les settings & mappings définis dans le body request de l'appel API de create index qui vont over-write ceux de template.

## II- Création d'un index-template:
- Au moment de création de template, il faut définir 3 choses principales: l'index_patterns & settings & mappings.
- L'appel est:
```javascript
    PUT _template/{name_template}
    {
        "index_patterns": ["........"],
        "settings": {
            .........
        },
        "mappings":{
            "properties":{
                .........
            }
        }
         
    }
  ```
- Exemple:
  - Si on lance l'appel suivant:
    ```javascript
        PUT _template/elastic-logs
        {
            "index_patterns": ["elastic-log-*"],
            "settings": {
                .........
            },
            "mappings":{
                "properties":{
                    .........
                }
            }
            
        }
    ```
   - Cet appel signifie qu'on a créé un template nommé **elastic-logs** qui sera appliqué automatiqument par ES sur tout les index dont le nom commence par **elastic-log-**


## III- Application d'un index-template à des index:
- Afin d'appliquer un template déjà défini sur un nouveau index, il suffit de lancer l'appel "PUT" ci-dessous, et en nommant ce nouveau index à créer avec un nom qui match l'index_patterns de notre template (sans définir ni les settings ni les mappings dans body request de cet appel "PUT"):

```javascript
    PUT elastic-log-sys-1
```
- Si on fait un appel "GET" de cet index **elastic-log-sys-1**, on peut remarquer que le settings & mappings de cet index correspond bien àceux de notre template **elastic-logs**:
```javascript
    GET elastic-log-sys-1
```
## IV- Index-template & Order:
### 1. Définition
- **Order** est un paramètre qu'on peut définir optionnellement au moment de création de template:
```javascript
PUT _template/elastic-logs
{
    "index_patterns": ["elastic-log-*"],
    "order":2,
    "settings": {
        .........
    },
    "mappings":{
        "properties":{
            .........
        }
    }
            
}
```
- Pourquoi on définit ce paramètre ???
    - Dans le cas où plusieurs templates peuvent potentiellement s'appliquer/matcher au meme index, => dans ce cas il faut prévoir d'ajouter/définir le paramètre **order** dans chaque template.
    => Dans ce cas: Les settings & mappings de tous les templates seront mergés/fusionnés, dans la configuration finale de notre index.
    - L'ordre de merge/fusion de ces settings & mappings sera determiné/controlé par la valeur de paramètre **order**:
      - Le template ayant **la plus faible valeur de paramètre order** sera appliqué le premier.
      - Le template ayant **la plus grande valeur de paramètre order** sera appliqué le dernier et donc va over-write les settings & mappings des autres templates dont l'**order** est plus inférieur.
### 2. Exemple:
- **1** Si on définit deux templates ayant le meme index_patterns mais avec deux valeurs différentes de paramètre **order**:
    - Template **logs** avec un **order** égal à 1:
    ```javascript
    PUT _template/logs
    {
        "index_patterns": ["elastic-log-*"],
        "order":1,
        "settings": {
            "number_of_shards":2,
            "number_of_replicas":0
        },
        "mappings":{
            "properties":{
                .........
            }
        }
                
    }
    ```
    - Template **elastic-logs** avec un **order** égal à 2:
    ```javascript
    PUT _template/elastic-logs
    {
        "index_patterns": ["elastic-log-*"],
        "order":2,
        "settings": {
            "number_of_shards":2,
            "number_of_replicas":1
        },
        "mappings":{
            "properties":{
                .........
            }
        }
                
    }
    ```
- **2** Maintenant si on lance une API request pour créer un nouveau index (dont le nom match le **index_patterns** de deux templates) :
  ```javascript
    PUT elastic-log-sys-1
  ```
    - **=>** Cet appel permet de créer un index nommé **elastic-log-sys1** selon le template ayant le plus grand **order**, çàd le template **elastic-logs**, car: (a) le nom de cet index match l'index_patterns de deux templates **logs** et **elastic-logs** (b) le template **elastic-logs**, ayant l'order le plus élévé, va over-write les settings & mappings de template **logs**

- **3** Si on lance une API GET pour recupérer notre index récement créé **elastic-log-sys-1** => on remarquera que les settings & mappings de cet index  correspond à ceux de template  **elastic-logs**:
  ```javascript
    GET elastic-log-sys-1
  ```

### 3. Règles:
- Si on veut définir deux ou plusieurs templates ayant le meme index_patterns:
    - Il faut que chaque template aura un nom de template différent de l'autre.
    - Il faut définir le paramètre **order**
