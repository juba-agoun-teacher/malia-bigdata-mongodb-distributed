# TP MongoDB: prise en main, pipeline d'agrégation et distribution

L'objectif de ce TP est de mettre en pratique le requêtage via une algèbre en
utilisant MongoDB.

Dans une **première partie**, on *prendra en main le shell MongoDB* ainsi que les requêtes de base. 

Dans une **seconde partie**, on prendra en main quelques
opérateurs d'aggrégation. 

Dans la **troisième partie** on combinera les opérations
d'aggrégation dans des _pipelines_ plus complexes.

**Nous allons dans un certain moment considérer un environnement distribué et tester sa résilience.**

> Pensez à rédiger un compte-rendu au format Markdown dans lequel vous
> indiquerez vos remarques ainsi que les différentes requêtes que vous aurez
> exécuté.

## 1. Connexion
L'architecture de MongoDB qu'on va utiliser dans ce début de TP suit cette architecture :  

<!-- ![archi](mongo_archi.png)(https://moodle.univ-lyon2.fr/pluginfile.php/468564/mod_folder/content/0/mongo_archi.png?forcedownload=1) -->

![archi](./img/archi_moodle.png )

Partie à faire en local, donc sur machine.

Tout d'abord! installer le serveur de base de données MongoDB depuis leur [site web](https://www.mongodb.com/try/download/community) 


### 1.2 Vérifier que le serveur MongoDB est installé

<!-- Pour faire le TP, trois choix possibles: utiliser le serveur mongodb commun, installer mongodb sur votre propre machine via des packages ou bien démarrer un serveur MongoDB via docker.  -->



Si Le serveur de base de données MongoDB est déjà installé,
Vous pouvez le verifier en allons voir dans le répertoire : 

> C:\Program Files\MongoDB\Server

Pour verifier si le serveur tourne sur votre machine, nous allons utiliser l'inviter de commande (CMD), suivez ses étapes:

- Appuyez sur sur les touches `Win + R` pour ouvrir la boîte de dialogue Exécuter.
- Tapez `cmd` et appuyez sur Entrée
- Dans l'invité de commande executez


```shell
sc query state= all | find "MongoDB"
```

Maintenant que vous avez vérifier que le serveur de base de données est en train de tourner sur votre machine, il vous faudra télécherger les données du  fichier [malia_db-mondgodb.zip][malia_db-mongodb.zip].

> Merci de ne pas oublier de décompreser le fichier .zip, c'est plus pratique ( ͡° ͜ʖ ͡°)

Pour effectuer les requêtes, on pourra utiliser le shell `mongosh`, le shell `mongo` ou bien l'interface graphique [MongoDB Compass][mongodb-compass] (*Pas de panique* 😰! *Nous allons expliquer cela!*).

Tout d'abord nous allons installer `mongosh` et pour cela il faut télécharger ce fichier d'installation [télécharger][mongosh.msi]

> ℹ️ L'installation de `mongosh` permet de utiliser le serveur de base de données MongoDB en ligne de commande. En d'autre termes, `mongosh` est conçue pour fournir une expérience utilisateur améliorée pour travailler avec la base de données MongoDB à partir de la ligne de commande. 
<!-- 
### 1.1 Connexion au serveur commun

Un serveur MongoDB est disponible sur `bd-pedago.univ-lyon1.fr`. Ce serveur
n'est disponible que depuis le réseau du campus: depuis les machine de TP ou
depuis le Wifi Eduroam. Il faut donc activer le VPN pour y avoir accès depuis
l'extérieur du campus ou depuis des réseaux Wifi comme Eduspot ou UCBL-Portail.

La base à laquelle on peut se connecter pour le TP est `mif04` avec le compte
`mif04`. Le mot de passe sera indiqué via tomuss.

On pourra adapter la commande suivante pour se connecter:

```shell
mongosh -u mif04 -p "remplacez-moi" mongodb://bd-pedago.univ-lyon1.fr/mif04
```

> Changer `remplacez-moi` par le mot de passe qui aura été communiqué

> Changer `mongosh` par `mongo` si la première commande n'est pas disponible. Si `mongo` n'est pas non plus disponible, télécharger [mongosh-1.1.7-linux-x64.tgz](https://downloads.mongodb.com/compass/mongosh-1.1.7-linux-x64.tgz) (utiliser `/tmp` au besoin car l'archive fait environ 60 Mo)

Pour [MongoDB Compass][mongodb-compass], utiliser la chaîne de connexion suivante, en remplaçant `remplacez-moi` par le mot de passe:

```
mongodb://mif04:remplacez-moi@bd-pedago.univ-lyon1.fr:27017/mif04?authSource=mif04&readPreference=primary&appname=MongoDB%20Compass&directConnection=true&ssl=false
```
 -->
### 1.2 Importer les données dans du serveur MongoDB
#### 1.2.1 Importer les données avec `mongoimport`
<u>Si vous souhaiter installer MongoDB dans vos ordianateurs personnelle</u>, il faut installer MongoDB Community Edition  (*Ce qui corespond au serveur de la base de données seulement*) via un système de packages, voir la [documentation d'installation][mongodb-install]. 

> ⚠️ Nous n'allons pas l'utiliser dans cet première partie du TP mais c'est bon a savoir que c'est possible de le faire (^.^)

Importer ensuite les données en utilisant l'outil `mongoimport` (qu'il faut installer au préalable) via la commande suivante (à exécuter dans le répertoire où le [zip][mif04-mongodb.zip] a été extrait):


<!-- 
# imorter les doonées depuis json file dans linux

for i in *.json; do mongoimport -c $(basename $i .json) --file=$i --type=json --jsonArray; done
-->


```shell
for %i in (*.json) do (set "collectionName=%~ni"  mongoimport --collection collectionName% --file "%i" --type json --jsonArray)
```
> ⚠️ Nous n'allons pas utiliser la commande précedente dans ce TP mais c'est bon a savoir que c'est possible de le faire (^.^)


#### 1.2.2 Importer les données avec l'interface graphique MongoDB Compass

Tout d'abord, vous devez installer l'outil que vous pouvez télécharger [MongoDB Compass][./img/mongodb-compass.exe]

Une fois le fichier télécharger, suiver les étapes par défaut pour l'installer. Vous devez trouver un raccourcis avec une icone ![icon_mongo](icon_mongo_compass.png) sur le bureau de votre machine. 

Ouvrez l'application compass, connectez-vous au serveur localhost. ⚠️ Ne changez pas les paramètres, utilisez le loclalhost et le port par défaut de compass.

![fen_compass](./img/fen_con_compass.png)

Vous êtes désormais connecter au serveur, vous avez sur votre coté gauche les bases de données par défauts et nécessaire de MongoDB. 

On va désormais créer une base de donnée, utiliser le boutons  **(+)**  à coté de _Databases_

![create_db](./img/create_db.png)

Nommez la base de données <mark style="background-color: #D9D9D9"> __malia_db__ </mark>  et la collection avec le même nom du premier fichier que vous avec télécheger depuis <mark style="background-color: #D9D9D9">_malia_db-mondgodb.zip_</mark>



On peut ensuite simplement lancer `mongosh` pour se connecter.
Pour [MongoDB Compass][mongodb-compass], laisser la chaîne de connexion vide.
<!-- 
### 1.3 Démarrage via docker.

Cette option nécessite d'avoir déjà Docker installé (Windows/MacOS: utiliser [Docker Desktop][docker], Linux: [installer via des packages][docker-linux]).

La commande suivante permet de lancer un serveur MongoDB en tâche de fond:

```shell
docker run --name mongodb -d -p 27017:27017 mongo:5.0.5-focal
```

Pour arrêter le serveur:

```shell
docker stop mongodb
```

Pour redémarrer un serveur arrêté:

```shell
docker start mongodb
```

Pour détruire le serveur:

```shell
docker rm --force mongodb
```

Pour importer les données (à lancer depuis le répertoire le zip a été décompressé):

```shell
for i in *.json; do cat $i | docker exec -i mongodb mongoimport --type=json --jsonArray -c $(basename $i .json) ; done
```

Pour lancer le shell:

```shell
docker exec -it mongodb mongosh
```

Pour [MongoDB Compass][mongodb-compass], laisser la chaîne de connexion vide.
 -->
### 1.4 Vérification de la connexion

Dans le shell mongo, vérifiez que vous êtes bien connecté en lançant la commande:

```
show collections
```

qui doit produire une sortie du type:

```
grades
neighborhoods
restaurants
zips
```

Cette sortie indique les collections disponibles dans la base courante.

Dans [MongoDB Compass][mongodb-compass], ces 4 collections doivent apparaître dans la base `mif04` ou dans la base `test`.


Si vous rencontrez une erreur de connexion de votre mongosh vers le serveur mongodb, utiliez cette commande au lieu  de `mongosh` : 
>  mongosh "mongodb://localhost:27017/mydatabase?serverSelectionTimeoutMS=5000"


## 2. Premières requêtes

Pour ces requêtes, on utilisera les collections `grades` et `zips`.
La syntaxe dans `mongosh` est `db.`_collection_`.`_methode_ où _collection_ est le nom de la collection à requêter (l'équivalent d'un `FROM` en SQL) et méthode est la manière de requêter la collection.

1. `db.collection.findOne()`: renvoie un document de la collection ([doc][findone]).
   > Pour chacune des collections `grades` et `zip`, récupérer un document, puis en déduire un type pour les éléments de cette collection.
2. `db.collection.find({})`: récupère les documents de la collection sans les filtrer ([doc][find]).
   > Récupérer quelques documents de grades et vérifier qu'ils sont conformes au type de la question précédente.
3. `db.collection.find({}).countDocuments()`: compte le nombre de résultats de la requête.
   > Donner le nombre de résultats de la requête précédente (résultat attendu: 280).
4. `db.collection.find({ `_field_`: `_value_` })`: récupère les documents dont le champ _field_ a la valeur _value_. Il est possible de spécifier plusieurs champs.
   > Récupérer les documents dont le `class_id` vaut `20` dans la collection `grades` (7 résultats).
5. `db.collection.find({ `_field_`: { `_op_`: `_value_` } })`: exprime une condition de sélection sur le champ _field_: _op_ est la fonctionde comparaison et _value_ la valeur à laquelle on veut comparer la valeur du champ ([doc des opérateur de sélection][query-selectors]).
   > Récupérer les documents dont le `class_id` est inférieur ou égal à `20`. L'opérateur inférieur ou égal se note `$lte`. (188 résultats)
6. `db.collection.find({ $expr: { `_arbre de syntaxe de l'expression en json_`} } )`: exprime une condition générique. La syntaxe est la représentation en json de l'expression ([doc][aggregation-expression]). Les champs sont représentés par la syntaxe `$champ` ou `$champ1.souschamp2`. Les arguments sont souvent donnés sous forme d'une liste (mais il faut vérifier la [documentation][aggregation-expression-operators] de chaque opérateur). Par exemple, `{ $lte: [ "$class_id", 20 ] }` est une expression indiquant que le champ `class_id` (via `"$class_id"`) est inférieur (`$lte`) à `20`.
   > Récupérer les documents pour lesquels le `student_id` est supérieur ou égal au `class_id` (188 résultats).
7. > Récupérer les documents dont le `class_id` est compris entre `10`et `20` (100 résultats). Faire une version avec un double filtre sur le champ `class_id` (via la syntaxe `{ `_field_`: { `_op1_`: `_value1_`, `_op2_`: `_value2_` } }`), puis une autre version avec un `$expr` contenant un `$and`.
8. `db.collection.find({`_condition_`}, { `_projection spec_` } )`: permet de choisir les champs de sortie de la requête ([doc][projection]). Il est possible de supprimer des champs, voir de créer de nouveaux champs en utilisant une expression comme ci-dessus.
   > Donner tous les documents de la collection en renommant le champ `class_id` en `ue` et le champ `student_id` en `etu`.
## 3. Prise en main de quelques étapes d'agrégation

En MongoDB, les requêtes complexes s'effectuent entre autres via le [pipeline d'agrégation][aggregation-pipeline]. Ce pipeline est constituées d'étapes de transformation successives appliquées à une collection et passées sous forme de liste à `db.`_collection_`.aggregate`. Dans cette partie, on va appréhender les effets de quelques opérateurs pris individuellement. Syntaxiquement, on spécifiera les requêtes comme suit:

```mongodb
db.grades.aggregate([
    { /* operateur */ : { /* specification */ }},
])
```

Pour compter le nombre de résultats, on pourra précéder comme suit:

```mongodb
db.grades.aggregate([
    { /* operateur */ : { /* specification */ }},
    { $count: "count" },
])
```

1. `{ $project: { `_projection spec_` }}`: applique une projection spécifiée comme pour le deuxième argument de `db.collection.find()` ([doc][$project]).
   > Donner tous les documents de la collection `grades` en y ajoutant un champ somme dont la valeur est la somme des champs `class_id` et `student_id`. on utilisera pour cela le pipeline d'agrégation avec un `$project`.
2. `{ $match: { `_find spec_` } }`: applique un filtre similaire au premier argument de `db.collection.find()` ([doc][$match]).
   > Dans la collection `zips` récupérer les documents ayant une population supérieure ou égale à `10000` (7634 résultats).
3. `{ $sort: {`_sort spec_` } }`: trie la collection selon les champs spécifiés ([doc][$sort]). Pour chaque champ, on indique s'il est trié par ordre croissant ou décroissant. Les champs sont listés par ordre d'apparition dans l'ordre lexicographique utilisé. Par exemple, sur la collection `zips`, `{ state: 1, pop: -1 }` inquera de trier par état (`state`), puis par population (`pop`) décroissante.
   > Dans la collection `grades`, renvoyer les documents triés par `class_id`, puis par `student_id` (par ordre croissant pour les 2 champs).
4. `{ $unwind: `_chemin_`}` : _chemin_ indique un champ, éventuellement dans un document imbriqué (via la syntaxe commençant avec `$`). Ce champ contient un tableau. L'opérateur produit un document pour chaque élément du tableau. Les documents produits sont identique au document de départ, sauf pour le champ désigné qui, au lieu du tableau, contient un élément de ce tableau.
   > Dans la collection `grades`, produire un document pour chaque élément du tableau contenu dans le champ `scores` (1241 résultats).
5. `{ $group: { _id: `_spec id_`, `_champ1_`: { `_agg1_`: `_expr1_` }, ...} }` ([doc][$group]): regroupe les documents selon la valeur _spec id_, qui est une expression utilisant la syntaxe des [expressions d'agrégation][aggregation-expression] (chemins avec `$`, [opérateurs d'expressions][aggregation-expression-operators], etc). On peut fabriquer des documents, ce qui permet de représenter des n-uplets pour utiliser des combinaisons de valeurs comme clés. Pour chaque groupe, produit un document avec la valeur de `_id`, ainsi que chaque champ supplémentaire (_champ1_, etc) avec la valeur calculée par l'opérateur d'agrégation correspondant (_agg1_, etc) à qui on aura passé le tableau des valeurs calculées par les expressions (_expr1_, etc). Les fonctions d'agrégation utilisables sont indiquées dans [la documentation][group-agg-operators]. Par exemple `{ $group: { _id: "$state", total: { $sum: "$pop" }}}` va créer un document pour chaque valeur du champ `state` avec un champ `total` qui sera la somme des champs `pop` des documents du groupe.
   > Dans la collection `zips`, donner pour chaque ville la population minimale (16584 résultats).
6. `{ $lookup: { from: `_coll_`, localField: `_champ local_`, foreignField: `_champ ext_`, as: `_champ out_` } }` ([doc][$lookup]): effectue une jointure avec `coll`. La jointure se fait sur une égalité entre le champ _champ local_ pour les documents venant du pipeline courant et le champ _champ ext_ pour les documents venant de la collection _coll_. Produit un document pour chaque document de la collection du pipeline, avec un champ supplémentaire (_champ out_) contenant le tableau des documents de _coll_ satisfaisant le critère de jointure.
   > Effectuer une jointure entre `grades` et `zips` sur les champs `student_id` et `pop`. Quel est le type du résultat ?

## 4. Mise en place d'un cluster MongoDB distribué sur Codespace

Dans cette partie, nous allons mettre en place une architecture MongoDB distribuée en réplica set (ensemble de réplication) sur GitHub Codespaces. Cette approche vous permettra d'expérimenter les concepts de distribution et de haute disponibilité en MongoDB.

### 4.1 Préparation de l'environnement Codespace

#### Étape 1 : Créer un Codespace blank

1. Accédez à [GitHub Codespaces](https://github.com/codespaces)
2. Cliquez sur **"Create codespace on main"** ou **"New codespace"**
3. Sélectionnez un template **blank** (recommandé: Ubuntu)
4. Attendez l'initialisation de l'environnement (quelques minutes)

Vous disposez maintenant d'un environnement Linux complet accessible via VS Code.

#### Étape 2 : Structure du projet

Créez l'architecture de dossiers suivante:

```
codespace-project/
├── docker-compose.yml
├── init-replica.js
├── data/
   ├── contacts.json
   ├── grades.json
   ├── neighborhoods.json
   ├── restaurants.json
   └── zips.json

```

### 4.2 Configuration du cluster MongoDB avec Docker Compose

Créez un fichier `docker-compose.yml` qui définit le cluster distribuée avec 1 nœud master et 3 nœuds workers:

```yaml
ersion: "3.8"

services:
  mongo1:
    image: mongo:7
    container_name: mongo1
    command: ["mongod", "--replSet", "rs0", "--bind_ip_all"]
    ports:
      - "27017:27017"
    networks:
      - mongo-net
    volumes:
      - mongo1_data:/data/db
      - ./init-replica.js:/init-replica.js

  mongo2:
    image: mongo:7
    container_name: mongo2
    command: ["mongod", "--replSet", "rs0", "--bind_ip_all"]
    networks:
      - mongo-net
    volumes:
      - mongo2_data:/data/db

  mongo3:
    image: mongo:7
    container_name: mongo3
    command: ["mongod", "--replSet", "rs0", "--bind_ip_all"]
    networks:
      - mongo-net
    volumes:
      - mongo3_data:/data/db

  mongo4:
    image: mongo:7
    container_name: mongo4
    command: ["mongod", "--replSet", "rs0", "--bind_ip_all"]
    networks:
      - mongo-net
    volumes:
      - mongo4_data:/data/db

networks:
  mongo-net:
    driver: bridge

volumes:
  mongo1_data:
  mongo2_data:
  mongo3_data:
  mongo4_data:
```

### 4.3 Initialiser le Replica Set

Créez un fichier `init-replica.js` pour initialiser le replica set:

```javascript
// init-replica.js
// Initialise le replica set avec les 4 nœuds

rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo1:27017" },
    { _id: 1, host: "mongo2:27017" },
    { _id: 2, host: "mongo3:27017" },
    { _id: 3, host: "mongo4:27017" }
  ]
})

```

### 4.4 Lancer le cluster

Dans le terminal Codespace, exécutez:

```shell
docker-compose up -d
```

Cela va démarrer les 4 conteneurs MongoDB. Attendez quelques secondes que tous les services soient prêts.

Pour vérifier que les conteneurs tournent:

```shell
docker-compose ps
```

Ensuite, initialisez le replica set en exécutant init-replica.js dans mongo1 :

```shell
# votre commande
```
> expliquer a en quoi consiste la commande précedente


Pour vérifier l'état du replica set on execute _rs.status()_ dans **mongo1**:

```shell
docker exec mongo1 mongosh --eval "rs.status()"
```


### 4.5 Vérifier la connexion avec mongosh

Pour vérifier la connexion au cluster:

1. **D'abord, installez mongosh localement sur Codespace**:

```shell
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt-get update
sudo apt-get install -y mongosh
```

2. **Testez la connexion avec mongosh**:

```shell
....
```

Une fois connecté, exécutez un teste de vérification en retrouvant toute les bases existantes:

```javascript
.....
```

### 4.6 Importer les données

Une fois le cluster en place et la connexion vérifiée:

1. **Installez mongoimport** (il est probable que ça ne marche pas du premier coup, essayer de resoudre le problème, ou appeler ⭐⭐l'enseignant⭐⭐ dans le pire des cas):

```shell
sudo apt-get install -y mongodb-database-tools
```

2. **Importez les données** (si vous avez les fichiers télécharger depuis __malia_db-mongodb.zip__ dans le dossier `data/`) faite le imprort de chaque fichier dans une collection differente de même nom:

```shell
....
```

3. **Vérifiez les données importées**:

```shell

```

### 4.7 Arrêter le cluster

Pour arrêter le cluster quand vous avez terminé:

```shell
docker-compose down
```

Pour supprimer complètement les volumes de données:

```shell
docker-compose down -v
```


## 5. Pipelines d'agrégation

_Pour un aspect visuel et pour eviter de tout faire sur le shell de codespace, on peut utiliser deux extenssions VScode: Mongo Playground (pour les pipelines) et s'aider avec MongodbVScode._

Dans cette partie, on va créer des pipelines d'aggrégation en enchaînant plusieurs opérateurs. Il faudra prévoir l'enchaînement des transformations à effectuer, et pour chaque transformation du pipeline, bien prévoir le type des documents du résultat afin de pourvoir configurer correctement les transformations suivantes.

**Exemple**: donner, par ordre de population décroissante, les états ayant plus de 10 000 000 d'habitants:

```mongodb
db.zips.aggregate([
    { $group: { _id: "$state",
                population: { $sum: "$pop" } } },
    { $match: { population: { $gte: 10000000 } } },
    { $sort: { population: -1 }},
])
```

Répondre aux questions suivantes via une requête utilisant un pipeline d'agrégation.

1. Calculer le nombre de notes de chaque type.
   _Indice:_ Commencer par extraire les notes individuellement avec `$unwind`.
   _Indice:_ On peut calculer un nombre d'occurrences en sommant la valeur `1` sur un groupe.
2. Donner pour chaque matière, la meilleure note d'examen.
3. Donner les dix villes les plus peuplées.
   _Indice:_ On pourra utiliser l'étape [$limit].
4. Donner la population moyenne des villes pour chaque état.
5. Donner pour chaque matière, les étudiants ayant une note d'examen supérieure à 50.
   _Indice:_ On pourra utiliser l'opérateur d'agrégation [\$push][$push] dans un [$group] pour constituer la liste.
6. Donner pour chaque étudiant et chaque matière sa note générale. La note générale est la moyenne de chaque type de note. S'il y a plusieurs notes d'un même type, on prendra d'abord la moyenne de ce type avant de l'utiliser pour calculer la moyenne avec les autres types.
   _Indice:_ Commencer par calculer pour chaque étudiant, chaque matière et chaque type de note sa moyenne.
7. Donner, pour chaque matière le type d'épreuve la mieux réussie. Le type d'épreuve la mieux réussie est celui ayant la meilleure moyenne, calculée sur l'ensemble des notes de ce type pour cette matière.
   _Indice:_ Un moyen de faire consiste à calculer la meilleure moyenne (via [\$max][$max]), tout en conservant l'ensemble des moyennes dans un tableau via [$push].

## 6. Résilence face aux pannes

Dans cette partie, nous allons simuler des pannes en arrêtant des nœuds du cluster pour vérifier que les données restent accessibles et que MongoDB gère correctement le failover (basculement automatique).

### 6.1 Vérifier l'état initial du cluster

Avant de commencer les tests, connectez-vous au cluster et vérifiez que tout fonctionne correctement:

```shell
docker exec mongo1 mongosh --eval "rs.status()" | head -50
```

Vérifiez aussi que les données sont accessibles:

```shell
docker exec mongo1 mongosh --eval "db.grades.countDocuments()" --quiet
```

### 6.2 Test 1: Arrêter un nœud worker (Secondary)

#### Arrêter mongo2

Arrêtez l'un des nœuds secondaires:

```shell
docker stop mongo2
```

Vérifiez que le conteneur est bien arrêté:

```shell
docker-compose ps
```

#### Tester l'accès aux données

Vérifiez que vous pouvez toujours accéder aux données via mongo1:

```shell
docker exec mongo1 mongosh --eval "db.grades.countDocuments()" --quiet
```

Le résultat doit être le même qu'avant.

#### Vérifier l'état du replica set

```shell
docker exec mongo1 mongosh --eval "rs.status()" | grep -A 2 "mongo2"
```

Vous devez voir que `mongo2` est marqué comme `UNREACHABLE` ou `DOWN`.

#### Relancer le nœud

Relancez le nœud arrêté:

```shell
docker start mongo2
```

Attendez quelques secondes et vérifiez que le nœud se reconnecte au cluster:

```shell
docker exec mongo1 mongosh --eval "rs.status()" | grep -A 2 "mongo2"
```

### 6.3 Test 2: Arrêter le nœud Primary (mongo1)

C'est le test le plus important! Nous allons arrêter le nœud primaire et observer le comportement du cluster.

#### Identifier le nœud Primary

```shell
docker exec mongo1 mongosh --eval "rs.status().members | map(m => ({host: m.host, state: m.stateStr}))"
```

ou plus simplement:

```shell
docker exec mongo1 mongosh --eval "rs.isMaster()"
```

#### Arrêter le nœud Primary

```shell
docker stop mongo1
```

#### Tester l'accès aux données depuis un autre nœud

Pendant que mongo1 est arrêté, tentez de vous connecter via un autre nœud et d'accéder aux données. **Attention:** Vous devez d'abord trouver quel est le nouveau PRIMARY parmi les nœuds restants.

```shell
docker exec mongo2 mongosh --eval "rs.status().members | map(m => ({host: m.host, state: m.stateStr}))"
```

Une fois que vous avez identifié le nouveau PRIMARY, vérifiez l'accès aux données:

```shell
docker exec mongo2 mongosh --eval "db.grades.countDocuments()" --quiet
```

#### Observer le failover

Pendant l'arrêt du nœud primary, vous pouvez observer les logs pour voir le processus de failover:

```shell
docker logs mongo2 | tail -20
```

#### Relancer le nœud Primary

```shell
docker start mongo1
```

Attendez quelques secondes et vérifiez que le cluster se stabilise:

```shell
docker exec mongo1 mongosh --eval "rs.status()" | head -100
```

### 6.4 Test 3: Arrêter plusieurs nœuds

#### Arrêter 2 nœuds secondaires

```shell
docker stop mongo2 mongo3
```

#### Tenter d'écrire une donnée

Essayez d'insérer un nouveau document dans la base de données:

```shell
docker exec mongo1 mongosh --eval "
db.grades.insertOne({
  student_id: 9999,
  class_id: 99,
  scores: [100, 100, 100]
})
"
```

Que se passe-t-il? Pourquoi?

#### Vérifier l'état du cluster

```shell
docker exec mongo1 mongosh --eval "rs.status().members"
```

#### Relancer les nœuds

```shell
docker start mongo2 mongo3
```

### 6.5 Test 4: Arrêter le Primary et le quorum

> ⚠️ **Attention:** Ce test va rendre le cluster non disponible!

#### Situation théorique

Nous avons 4 nœuds. Pour avoir un quorum (majorité), on a besoin d'au moins 3 nœuds. Si on arrête 2 nœuds dont le PRIMARY:

```shell
docker stop mongo1 mongo2
```

#### Vérifier l'indisponibilité

Essayez de vous connecter via mongo3:

```shell
docker exec mongo3 mongosh --eval "db.grades.countDocuments()" --quiet
```

Vous devez obtenir une erreur car il n'y a pas de quorum (2 nœuds sur 4 ne suffisent pas).

#### Vérifier le statut

```shell
docker exec mongo3 mongosh --eval "rs.status()" 2>&1 | head -30
```

#### Restaurer le cluster

```shell
docker start mongo1 mongo2
```

Attendez quelques secondes et vérifiez que le cluster se rétablit:

```shell
docker exec mongo1 mongosh --eval "db.grades.countDocuments()" --quiet
```

### 6.6 Questions de synthèse

Répondez aux questions suivantes dans votre compte-rendu:

1. **Disponibilité des données**: Quand est-ce que les données restent accessibles en lecture et en écriture après une panne?

2. **Failover**: Combien de temps prend le processus de failover (élection d'un nouveau PRIMARY)?

3. **Quorum**: Pourquoi est-ce qu'avoir un quorum est important? Qu'arriverait-il avec 2 nœuds au lieu de 4?

4. **Données perdues**: Avez-vous observé une perte de données lors des pannes? Pourquoi (ou pourquoi pas)?

5. **Configuration**: Comment pourrait-on améliorer la résilience du cluster (nombre de nœuds, localisation géographique, etc.)?




[mongodb-compass]: https://www.mongodb.com/products/compass
[mongodb-compass.exe]:https://downloads.mongodb.com/compass/mongodb-compass-1.40.4-win32-x64.exe
[mongodb-install]: https://docs.mongodb.com/manual/installation/#mongodb-installation-tutorials
[docker]: https://www.docker.com/get-started
[docker-linux]: https://docs.docker.com/engine/install/#server
[malia_db-mongodb.zip]: ./malia_db-mongodb.zip

[malia_db-mongodb.zip]: ./malia_db-mongodb.zip



[findone]: https://docs.mongodb.com/manual/reference/method/db.collection.findOne/
[find]: https://docs.mongodb.com/manual/reference/method/db.collection.find/
[query-selectors]: https://docs.mongodb.com/manual/reference/operator/query/#query-selectors
[aggregation-expression]: https://docs.mongodb.com/manual/meta/aggregation-quick-reference/#std-label-aggregation-expressions
[aggregation-expression-operators]: https://docs.mongodb.com/manual/meta/aggregation-quick-reference/#operator-expressions
[projection]: https://docs.mongodb.com/manual/reference/method/db.collection.find/#std-label-find-projection
[aggregation-pipeline]: https://docs.mongodb.com/manual/aggregation/#std-label-aggregation-framework
[$project]: https://docs.mongodb.com/manual/reference/operator/aggregation/project/#mongodb-pipeline-pipe.-project
[$match]: https://docs.mongodb.com/manual/reference/operator/aggregation/match/#mongodb-pipeline-pipe.-match
[$sort]: https://docs.mongodb.com/manual/reference/operator/aggregation/sort/
[$group]: https://docs.mongodb.com/manual/reference/operator/aggregation/group/
[group-agg-operators]: https://docs.mongodb.com/manual/reference/operator/aggregation/group/#std-label-accumulators-group
[$lookup]: https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/
[$limit]: https://docs.mongodb.com/manual/reference/operator/aggregation/limit/
[$max]: https://docs.mongodb.com/manual/reference/operator/aggregation/max/#mongodb-group-grp.-max
[$push]: https://docs.mongodb.com/manual/reference/operator/aggregation/push/#mongodb-group-grp.-push
