# Re-indexation
## I- Définition:
- La **re-indexation** de data, présente dans un ancien index, vers un nouveau index = copier tout (ou une partie) de data de cet ancien index vers le nouveau index.
- Re-indexation permet de re-indexer/copier la totalité (ou une partie) de data d'un ancien index vers le nouveau index (qui est déjà initialisé)
## II- Pourquoi/Quand on fait re-indexation ?
- On fait la re-indexation de data dans deux cas:
    - **1er cas** Si on souhaite ajouter un nouveau field ou modifier un field déjà existant dans le mappings d'index (par exp: le champs "age" était non-searchable, etvon souhaite le rendre searchable).
    - **2ème cas** Reindexation pour modifier le nombre de shards:
    Si le volume de data dans l'ancien index est devenu très grand et le nb de shards de cet index est trop faible (2 shards par exp):
    Dans ce cas, la solution est d'augmenter le nb de shards (afin de partager ce grand volume de data sur bcp plus de shards). Mais, on sait déjà que le nombre de shards, dans un index déjà créé, est non modifiable.
    C'est ici où on fait la **re-indexation**: on re-indexe le data de l'ancien index (ayant le faible nombre de shards) vers un nouveau index (ayant un nombre plus élévé de shards).
    Pour faire cela, on passe par deux étapes: (1) créer/initialiser un nouveau index avec les nouveaux settings (nombre de shrds..) et les nouveaux mappings (nvx fields ou fields modifiés). (2) re-indexer les data de l'ancien index vers ce nouveau index déjà initialisé.
    Donc, la **re-indexation** ne copie pas les fields de settings & mappings, mais plutot les valeurs de ces fields (dans les settings & mappings déjà créés dans le nouveau index)
## III- API Request de Re-indexation:
- Il y a deux manières pour faire l'iopération de re-indexation de data:
### 1. Opération de re-indexation basique:
- Cette request permet de re-indexer tout le data dés l'index source (nommé "index_1" dans l'exemple) vers l'index destinataire (nommé new_index_1 dans l'exemple):
  ```javascript
    POST _reindex
    {
        "source": {
            "index": "index_1"
        },
        "dest": {
            "index": "new_index_1"
        }
         
    }
  ```
### 2. Opération de re-indexation avancée (avec l'option filter):
- Cette request permet de re-indexer une partie de data dés l'index source (nommé "index_1" dans l'exemple), dont la valeur de field "name" est égal à "Heithem", vers l'index destinataire (nommé new_index_1 dans l'exemple):
  ```javascript
    POST _reindex
    {
        "source": {
            "index": "index_1",
            "query":{
                "match":{
                    "name": "Heithem"
                }
            }
        },
        "dest": {
            "index": "new_index_1"
        }
         
    }
  ```
## IV- Le paramètre "version_type":  
### 1- Problématique:
- Si au moment de reindexation, on se rend compte qu'un doc (d'id=1) de l'ancien index existe déjà dans l'index destinataire (çàd: il y a déjà un autre doc dans l'index dest. avec l'id=1).
- Comment ES va réagir dans ce cas:
  - **1** il va màj le doc d'id=1 dans l'index destinataire (over-write ce doc s'il existe dans l'index destinataire)
  - **2** ou il va l'ignorer (si le doc existe on ne le reindexe pas)
  - **3** ou il va reindexer que les docs non existants.
- La réaction d'ES (1, 2 et 3) sera décidée grace au propriété "**version_type**", qui sera définie au moment de re-indexation request.
### 2- Version_type = "internal" OU Version_type empty (non spécifié):
- Dans ce cas **ES is going to blindly dump documents int the target index over-writing any that happen to have the same id*
- Si le doc existe déjà dans l'index dest. (çàd son id existe déjà dans l'index dest): 
    - ES va updater/màj TOUT ces docs, peu importe leur version dans l'index destinataire.
    - Et ES va incrémenter la version de ces docs
### 3- Version_type = "external":
- Dans ce cas **ES creates any docs that are missing (whose _id doesn't exist yet in the target index)*
- Si le doc n'existe pas encore dans l'index dest. (çàd son id n'est pas encore présent dans l'index dest.):
    -  ES crée ce doc dans le nouveau index
- Si le doc existe déjà (dont l'id existe déjà dans l'index target):
    - ES va tout d'abord regarder la version de ce doc dans l'index dest. et si cette version est plus ancienne que celle de doc dans l'index source, ES va MAJ ce doc et va préserver la meme version de l'index source (voir que c(='est la version la plus récente))