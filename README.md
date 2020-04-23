# ElasticSearchTutorials
## Sommaire
* [Introduction et concepts de base d'Elastic Stack](https://github.com/HeithemLejmi/ElasticSearchTutorials/blob/feat/create_index_docs/Introduction_to_ES.md)
* [Creation & Update d'un index](https://github.com/HeithemLejmi/ElasticSearchTutorials/blob/feat/create_index_docs/Creation_of_index.md)
* [Opérations de document (indexation, update, search, delete)](https://github.com/HeithemLejmi/ElasticSearchTutorials/blob/feat/create_index_docs/Operations_doc.md)
* [Index Alias](https://github.com/HeithemLejmi/ElasticSearchTutorials/blob/feat/create_index_docs/Operations_index-alias.md)
* [Index Template](https://github.com/HeithemLejmi/ElasticSearchTutorials/blob/feat/create_index_docs/Operations_index-template.md)
* [Re-index](https://github.com/HeithemLejmi/ElasticSearchTutorials/blob/feat/create_index_docs/Operations_re-index.md)

### ElasticSearch sur Ubuntu:
#### Installer ElasticSearch:
Afin d'installer ElasticSearch sur ubuntu, il faut suivre les [étapes suivantes](https://www.fosslinux.com/6084/how-to-install-elk-stack-on-ubuntu-18-04.htm):

- Téléchargez Elasticsearch en utilisant le public signing key:
> sudo wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

- Installez le package apt-transport-https (Debian based distros a besoin de ça):   

> sudo apt-get install apt-transport-https

- Ajoutez la repository:
> echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list

- Mettez à jour la repo list et installe package:
> sudo apt-get update

> sudo apt-get install elasticsearch

- Modifiez le fichier de configuration “elasticsearch.yml”:

> sudo nano /etc/elasticsearch/elasticsearch.yml

- Une fois le fichier "elasticseearch.yml" est ouvert, Décommentez “network.host” et “http.port”. et Suivez la configuration suivante:
```
network.host: localhost
http.port: 9200
```
Ensuite, enregistrez le fichier "elasticseearch.yml" et quittez: (CTRL+X -> Y ou O -> Enter).

- Pour vous assurer qu'ElasticSearch fonctionne de manière transparente, activez-le au démarrage:
> sudo systemctl enable elasticsearch.service

#### Démarrer ElasticSearch:
- Afin de démarrer ElasticSearch sur ubuntu, lancez la commande suivante:
> sudo systemctl start elasticsearch.service

- ElasticSearch est lancé en local sur le port **9200**:
```
http://localhost:9200
```

- Pour arreter ElasticSearch:
> sudo systemctl stop elasticsearch.service

### Kibana sur Ubuntu:
#### Installer Kibana:
- Commençons maintenant à installer Kibana :
> sudo apt-get install kibana

- Modifions les paramètres de Kibana:
> sudo nano /etc/kibana/kibana.yml
- Une fois le fichier de configuration "kibana.yml" est ouvert, décommentez les lignes suivantes:
```
server.port: 5601
server.host: "localhost"
elasticsearch.url: "http://localhost:9200"
```
- Enregistrez et quittez le fichier.

- Activez Kibana au démarrage:
>sudo systemctl enable kibana.service

#### Lancer Kibana:
- Démarrez le service Kibana grace à la commande suivante:
>sudo systemctl start kibana.service

- Maintenant, Kibana est lancé en local sur le port **5601**:
```
http://localhost:5601/app/kibana
```

- Pour arreter Kibana:
> sudo systemctl stop kibana.service