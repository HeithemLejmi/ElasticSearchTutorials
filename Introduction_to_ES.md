# Introduction & Concepts de base

## I- Composition d'Elastic Stack:
- Il est composé de : Elasticsearch + Logstash + Kibana
- Logastash: responsable d'alimenter l'Elastic search avec le data, il s'occuper d'extraire/filtrer/traiter les datas venant de plusieurs sources de data (logs d'apache, restful API, kafka, ..) et les pousser dans Elastic search.
- Elastic search est le core et la base d'ELK stack.. il réçoit le data de Logstach et le visualise sur Kibana. Elastick search stocke le data et donne des fonctionnalités de recherche à travers ce data (Elastick search, joue à la fois le role de base de données (no sql) et le moteur de recherche)
- Kibana est un outil de visualistation et l'UI de l'ELK stack


## II- Concepts de base de l'organisation de data dans Elastic Search:
Les Data, en ES, sont contenus dans un index:
### A- Disposition logique de Data dans l'index: Index est composé des documents
#### 1) Index:
- c'est une collection de documents ayant des caractéristiques (fields) similaires...
- En analogie avec les BDR, un undex représente un tableau, et un doc dans l'index représente un row dans ce table (index).
- un index est caractérisé par son nom: 
---> qui doit etre en lettres miniscules 
---> le nom d'index est utilisé pour faire réf à cet index, dans toutes les opérations:
* indexation des documents
* search 
* update/delete des documents.. 

#### 2) Document:
- c'est l'information de base (unité de base) dans ES, qu'on peut l'indexr (le mettre en index), par exp:
--> dans un index de clients, un document représente un seul client,....
- le doc est exprimé (au moment d'indexation / recherche / update ...) sous format de json.


Donc, en ES: les datas est organisés sous forme des documents contenus dans un index.
ES est schemaless (contrairement au BDR), çàd qu'il y a pas besoin de définir/etre engagé à un schema pour notre data => c'est pas la peine de définr un schema pour nos doc ou notre index (mais c'est possible qd mm de définir le mapping de l'index et de ses docs).

### B- Disposition physique de Data dans l'index: Index est composé des shards et des replicas
=> un index est composé d'au moins un shard (1 ou plusieurs shards), et chaque shard peut avoir zero ou plusieurs replicas. 
Remarques:
* on ne peut pas créer un index sans aucun shard, ce pourquoi on a dit qu'un index doit etre composé d'au moins un shard.
* Les datas contenus dans l'index sera distribé/partagé sur ses shards, çàd si l'index a un seul shard, cela veut dire que ce shard contient le 100% de data de son index.
* un replica est une copie d'un shard

#### 1) Shard
- Chaque shard est considéré comme fully-functional and independent index qui peut etre hébergé dans n'importe quelle noeud (serveur) de notre cluster.
- nb de shards, sur lequels est partagé notre index, est défini au moment de creation de cet index (le nb de shards est non modifiable, une fois l'index est créé, on ne peut plus modifier son nobmre de shards)

#### 2) Avantage d'utilisation de shards:
- Les shards nous permettent de faire de up-scalling: partager horizontallement le volume total de data de l'index sur plusieurs shards.
- la mise de place de data sur plusieurs shards, nous permet de augmenter la performance et le débit des operations effectuées sur ce data: parallèliser les operations (sur plusieurs shards), par exp faire la recherche sur des plusieurs petits shards (contenant chacun une partie de data) sera plus rapide et efficace que faire la recherche sur un seul gros volume de data centralisé dans un seul shard representant la totalité de l'index.

#### 3) Avantage de replicas:
- la présence de replica fournit une haute disponibilité de data (sur plusieurs noeuds) et garantit la non-perte de data en cas de panne de serveur (noeud) sur lequel se trouve le shard (d'origine de ce replicas):
--> Remarque: On ne met jamais le shard et son propre replica sur le meme noeud, l'idée est d'éviter les pannes:
par exemple on a un shard A sur un noeud 1 et son replica A1 sur un noeud 2, si le noeud 1 ne marche plus, on ne perd pas nos data sur le shard A, car on a une copie déjà sur le replica A1 sur un autre noeud/

- la présence de replicas nous permet de faire de la mise à l'échelle de débit et volume de nos opérations sur le data: voir que les opérations de recherche (par exp) peuvent etre faites en parallèle sur tous les replicas.