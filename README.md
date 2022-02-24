# Introduction

Dans ce TP, vous allez installer la suite ELK (Elasticsearch, Logstash, Kibana) dans sa version open source : **OpenSearch** (Nouveauté 2022). Le coeur de cette suite est OpenSearch (fork Elasticsearch), un moteur de recherche puissant qui est de plus en plus utilisé dans l’industrie ; non seulement pour ses capacités de recherche mais aussi pour ses fonctionnalités de Business Intelligence (BI).

Vous allez travailler sur des données factices représentant des comptes utilisateurs. A l’issu de ce TP, vous serez capable de :
- Déployer un environnement ELK en quelques minutes ;
- Ingérer des documents JSON (semi-structurés) ;
- Construire et exécuter des requêtes ;
- Fabriquer des visualisations permettant d’explorer vos données grâce à OpenDashboards (fork Kibana) ;
- Ingérer d’autres formats de données grâce à logstash.

Evaluation : L’évaluation portera sur un rapport d’un 10aine de pages dans lequel vous indiquerez _honnêtement_ les réponses aux questions à partir du sous titre Requêtage simple. 
Le rapport est à soumettre sur Moodle avant la date indiquée.


# Installation de la suite ELK

Créer le dossier “TP-ELK”
A la racine, créer le fichier docker-compose.yml  qui contiendra toutes les instructions à la mise en place de l’environnement ELK (3 containers, un pour OpenSearch, un pour logstash et un pour OpenSearch-dashboards). Configuration :

## Images

```
opensearchproject/opensearch:1.2.4
opensearchproject/logstash-oss-with-opensearch-output-plugin:7.13.4
opensearchproject/opensearch-dashboards:1.2.0
```

## Ports

```
9200 (opensearch/http), 5601 (opensearch-dashboards/http), 5000 (logstash-oss/tcp)
```

## Réseau

```
bridge
```

Aidez-vous de la documentation https://opensearch.org/downloads.html#docker-compose pour créer votre fichier docker-compose.yml.

Exécutez 

```
sysctl -w vm.max_map_count=262144 
```

pour qu'OpenSearch se lance correctement.

Exécuter docker-compose up pour construire l’environnement. Vérifier le bon fonctionnement de l’environnement en inspectant les logs de docker-compose et en vous connectant à l’URL http://localhost:9200

# Partie 1 - Gestion de données simples
## Indexation de données 
Télécharger le jeu de données sur https://download.elastic.co/demos/kibana/gettingstarted/accounts.zip
Indexer les données dans OpenSearch grâce à la commande suivante :

```
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/bank/_bulk?pretty' --data-binary @accounts.json
```

Toutes les données sont alors envoyées dans l’index nommé bank et le type account.


### Prise en main d'OpenSearch
Rendez-vous à l’URL http://localhost:5601 puis dans Dev Tools

A l’aide de la documentation disponible au lien ci-dessous, vous allez écrire les requêtes permettant de répondre à différentes questions.
https://www.elastic.co/guide/en/elasticsearch/reference/current/docs.html

## Requêtage simple 

### INDEX et UPDATE API
- Après avoir étudié la structure d’un document, ajouter un nouveau compte dont l’ID est 10001
- Mettre à jour le compte précédent en modifiant l’adresse

### DELETE API
- Supprimer le compte précédemment créé
- Supprimer tous les comptes de la ville de Nicholson (Utiliser _delete_by_query)

### GET API
- Obtenir le compte dont l’ID est 2
- Idem mais récupérer uniquement le source (champs _source)
- Idem mais en ne sélectionnant que le nom et le prénom (firstname, lastname)

### SEARCH API
- Retrouver tous les comptes
- Retrouver tous les comptes dont la ville (champs city) est Belvoir
- Retrouver tous les comptes dont la ville (champs city) est Belvoir ET l’employeur Xurban

## 
- Visualisation des données avec OpenSearch Dashboards
- Rendez-vous sur l’onglet Visualize et créer les visuels suivants :

- Métrique affichant la somme des soldes de tous les comptes. Sauver le graphique.
- Graphique barre affichant la moyenne des soldes (champ balance) selon l’état (champ state). Sauver le graphique.
- Nuage de mots représentant les villes dans lesquelles il y a le plus de comptes. Sauver le graphique.

Rendez-vous ensuite sur l’onglet Dashboard puis : 

- Réaliser un tableau de bord regroupant les 3 visuels précédents.
- Exécuter une requête dans la barre de recherche situé au dessus et remarquez que les visuels se mettent automatiquement à jour.

# Partie 2 - Gestion de documents textuels

Dans cette partie nous allons travailler avec des données principalement textuelles et nous allons palper la puissance d'OpenSearch en tant que moteur de recherche plein texte.

## Indexation de données
- Télécharger le jeu de données https://download.elastic.co/demos/kibana/gettingstarted/shakespeare_6.0.json
- Indexer les données dans OpenSearch grâce à la commande suivante :

```
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/shakespeare/doc/_bulk?pretty' --data-binary @shakespeare_6.0.json
```

## Recherches plein texte
OpenSearch est un puissant moteur de recherche capable de gérer des millions de documents sur une unique machine et plusieurs milliards lorsqu’il est distribué. 

L’objectif des questions ci-dessous est de se rendre compte de ses capacités de recherche et de la rapidité à laquelle les réponses sont données.
URI SEARCH
Documentation : https://www.elastic.co/guide/en/elasticsearch/reference/current/search-uri-request.html

- Rechercher les documents contenant le terme KING dans les champs text_entry OU playname. Accordez deux fois plus d’importance aux documents qui contiennent le terme dans le champ play_name (astuce : KING^2).
- Rechercher les documents où l’orateur (champ speaker) CAESAR parle de Brutus (champ text_entry)
- Rechercher les documents où l’orateur(champ speaker) CAESAR ne parle PAS de Brutus (champ text_entry)
- Rechercher les documents répondant à la requête caesar brutus calpurnia
- Selon vous, pourquoi le cinquième document, qui contient tous les termes, n’est pas en première position ?
- Modifier la requête pour que seul le cinquième document réponde.
- Rechercher les documents répondant à la requête cesar (la faute est volontaire)
- Pourquoi aucun document ne répond ?
- Essayez maintenant avec la requête cesar~ 
- En déduire le rôle de l’opérateur ~

### AGGREGATION API

Trouver le nombre total de pièces (champ play_name)
En une requête, calculer le nombre de lignes (champ line_id) pour chaque pièce (champ play_name)

# Partie 3 - Pour aller plus loin : Logstash
Logstash est un formidable outil pour le traitement et l’ingestion de données, notamment dans Elastic search.

- Etudier logstash sur la page https://opensearch.org/docs/latest/clients/logstash/index/ 
- Télécharger la carte des hôtels classés en IDF ici https://www.data.gouv.fr/fr/datasets/la-carte-des-hotels-classes-en-ile-de-france-idf/
puis écrire une configuration Logstash pour ingérer ces données dans OpenSearch
- Développez un tableau de bord sur ces données. Ce tableau devra comporter une carte géographique des hôtels classés en fonction de leur nombre d’étoiles.

