# Notes TP4

**Auteur :** Louis AMEDRO, BUT3 APP

---

## Q1

_Le script utilise `MERGE` et non `CREATE`. Que se passerait-il si vous le lanciez deux fois avec `CREATE` ? À quoi servent les trois `CREATE CONSTRAINT` du début ?_

On utilise `MERGE` pour éviter les doublons et les erreurs. `CREATE CONSTRAINT` sert à créer avec une contrainte, ici, les joueurs, les jeux et les guildes sont uniques.

## Q2

_Combien de résultats pour chacune ? Ouvrez le script d'import et expliquez l'écart. Laquelle est correcte, et **pourquoi aucune erreur ne vous a prévenu** ?_

- 2 résultats pour la première requête (`AMI_DE` sans flèche).
- 1 résultat pour la deuxième requête (`AMI_DE` avec flèche).

La deuxième est correcte car les flèches indiquent la direction des relations entre le joueur `Krayz` et ses amis directement.

Il n'y a pas eu d'erreur car le script utilisait `MERGE` et non `CREATE`.

## Q3

_Remplacez `count(DISTINCT ami)` par `count(*)` : que compte chacune des deux versions ? Passez ensuite de `*1..2` à `*1..4` : combien de lignes changez-vous, et combien de CTE auriez-vous ajoutées en SQL ?_

```cypher
MATCH (moi:Joueur {pseudo:'Nova'})-[:AMI_DE*1..2]-(ami:Joueur)-[:POSSEDE]->(j:Jeu)
WHERE ami <> moi AND NOT (moi)-[:POSSEDE]->(j)
RETURN j.titre, count(*) AS nbAmis
ORDER BY nbAmis DESC
```

Les deux version compte le nombre d'amis de Nova qui possèdent chacun des jeux.

```json
[
  {
    "j.titre": "Valorant",
    "nbAmis": 2
  },
  {
    "j.titre": "Counter-Strike 2",
    "nbAmis": 2
  },
  {
    "j.titre": "Baldur's Gate 3",
    "nbAmis": 2
  },
  {
    "j.titre": "Rocket League",
    "nbAmis": 1
  },
  {
    "j.titre": "Mario Kart 8 Deluxe",
    "nbAmis": 1
  },
  {
    "j.titre": "Terraria",
    "nbAmis": 1
  },
  {
    "j.titre": "Vampire Survivors",
    "nbAmis": 1
  }
]
```

- La version avec `count(*)` compte le nombre d'amis uniques (joueurs distincts) qui possèdent le jeu. (Si un même ami est lié par plusieurs chemins, il n'est compté qu'une seule fois.)

- La version avec `count(DISTINCT ami)` compte le nombre total de lignes ou de chemins bruts renvoyés par le `MATCH`. (Si un ami possède le jeu et qu'il existe plusieurs chemins pour l'atteindre, alors, chaque chemin est compté. Ce qui peut fausser le nombre réel d'amis.)

## Q4

_Pourquoi \_Stardew Valley_ n'apparaît-il jamais dans les recommandations de Nova ?\_

Car Nova possède déjà Stardew Valley, et la requête filtre les jeux déjà possédés.

## Q5

_Le Browser est sur le port 17474, et l'application utilise 17687. Pourquoi deux ports pour le même serveur ?_

On utilise deux port car le port `17474` est le port par défaut pour le Browser (l'interface graphique web de Neo4j), et le port `17687` est le port par défaut pour la connexion à la base de données Neo4j directement.

## Q6

_Le pseudo est passé en paramètre (`$pseudo` + `new { pseudo }`), mais la profondeur est **interpolée** dans la chaîne. Pourquoi ? Quelle précaution le code prend-il, et que risquerait-on sans elle ?_

La profondeur doit être interpolée car les paramètres dynamiques ne sont pas acceptés pour définir les longueurs de chemins variables (\*1..2). Le code doit donc impérativement valider et convertir cette valeur en nombre entier strict pour contrer une faille d'injection Cypher, qui permettrait sinon à un utilisateur malveillant d'exécuter du code arbitraire ou destructeur sur la base de données.

[pas fini partie 1, passage à la partie 2]

## Q8

_Pourquoi les guillemets doubles ? Que répond PostgreSQL si vous écrivez `SELECT * FROM players;` ?_

Les guillemets doubles sont nécessaires car les noms de tables et de colonnes ont été créés en minuscules, mais PostgreSQL les considère comme sensibles à la casse lorsqu'ils sont entourés de guillemets. Sans guillemets, il cherche "players" (en minuscules), qui n'existe pas (seul "Players" existe).

PostgreSQL répond :

```bash
ERROR:  relation "players" does not exist
LINE 1: SELECT * FROM players;
                      ^
```

## Q9

_Que garantit la ligne `UNIQUE ("PlayerId", "Titre")` ? Et la contrainte `coins_positifs` ? Essayez : `UPDATE "Players" SET "Coins" = -1 WHERE "Pseudo" = 'Ombre';`_

Elle garantit qu'un joueur ne peut pas acheter le même jeu deux fois (unicité).

La contrainte `coins_positifs` empêche que la valeur dans la colonne "Coins" soit négative.

```bash
ERROR:  new row for relation "Players" violates check constraint "coins_positifs"
DETAIL:  Failing row contains (3, Ombre, -1).
```

## Q10

_Relevez le `modifiedCount` de chacune des trois commandes. Combien de documents avez-vous réécrits en entier ? Qu'aurait-il fallu faire en SQL ? Et si on lançait la **première** commande **en dernier** ?_

```bash
// La première commande
modifiedCount: 12

// La deuxième commande
modifiedCount: 8

// La troisième commande
modifiedCount: 4
```

J'ai donc réécrit 12 documents pour la première commande, 8 pour la deuxième et 4 pour la troisième.

En SQL, j'aurais utilisé un UPDATE avec une condition.

Si on lançait la première commande en dernier, le résultat serait le même. Les commandes se font de manière indépendante, il n'y a pas de dépendance entre elles.

## Q11

_La valeur en cache contient-elle les prix ? Pendant combien de temps, au plus, l'API servira-t-elle un catalogue sans prix ? Quelle commande Redis force la mise à jour **tout de suite** ? (Faites-la.)_

```bash
127.0.0.1:6379> TTL cache:catalogue:tous
(integer) 43
127.0.0.1:6379> GET cache:catalogue:tous
"[{\"Id\":\"6ac744343d2ce1ef135019c6\",\"Titre\":\"Hades II\",\"Genre\":\"Rogue-like\",\"Note\":4.7,\"AnneeSortie\":2024,\"Plateformes\":[\"PC\",\"Switch\"],\"Tags\":[\"rogue-like\",\"action\",\"solo\"]},{\"Id\":\"6ac744343d2ce1ef135019c7\",\"Titre\":\"Stardew Valley\",\"Genre\":\"Simulation\",\"Note\":4.8,\"AnneeSortie\":2016,\"Plateformes\":[\"PC\",\"Switch\",\"PS5\",\"Mobile\"],\"Tags\":[\"pixel art\",\"d\\u00E9tente\",\"coop\\u00E9ratif\"]},{\"Id\":\"6ac744343d2ce1ef135019c8\",\"Titre\":\"Valorant\",\"Genre\":\"FPS\",\"Note\":4.3,\"AnneeSortie\":2020,\"Plateformes\":[\"PC\"],\"Tags\":[\"comp\\u00E9titif\",\"tir\",\"h\\u00E9ros\"]},{\"Id\":\"6ac744343d2ce1ef135019c9\",\"Titre\":\"Baldur\\u0027s Gate 3\",\"Genre\":\"RPG\",\"Note\":4.9,\"AnneeSortie\":2023,\"Plateformes\":[\"PC\",\"PS5\",\"Xbox Series\"],\"Tags\":[\"narratif\",\"tour par tour\",\"coop\\u00E9ratif\"]},{\"Id\":\"6ac744343d2ce1ef135019ca\",\"Titre\":\"Vampire Survivors\",\"Genre\":\"Rogue-like\",\"Note\":4.4,\"AnneeSortie\":2022,\"Plateformes\":[\"PC\",\"Switch\",\"Mobile\"],\"Tags\":[\"rogue-like\",\"arcade\",\"solo\"]},{\"Id\":\"6ac744343d2ce1ef135019cb\",\"Titre\":\"Rocket League\",\"Genre\":\"Sport\",\"Note\":4.3,\"AnneeSortie\":2015,\"Plateformes\":[\"PC\",\"PS5\",\"Xbox Series\",\"Switch\"],\"Tags\":[\"comp\\u00E9titif\",\"voiture\",\"arcade\"]},{\"Id\":\"6ac744343d2ce1ef135019cc\",\"Titre\":\"Terraria\",\"Genre\":\"Bac \\u00E0 sable\",\"Note\":4.7,\"AnneeSortie\":2011,\"Plateformes\":[\"PC\",\"Switch\",\"PS5\",\"Mobile\"],\"Tags\":[\"exploration\",\"construction\",\"coop\\u00E9ratif\",\"sandbox\"]},{\"Id\":\"6ac744343d2ce1ef135019cd\",\"Titre\":\"Counter-Strike 2\",\"Genre\":\"FPS\",\"Note\":4.5,\"AnneeSortie\":2023,\"Plateformes\":[\"PC\"],\"Tags\":[\"comp\\u00E9titif\",\"tir\",\"\\u00E9quipe\"]},{\"Id\":\"6ac744343d2ce1ef135019ce\",\"Titre\":\"Mario Kart 8 Deluxe\",\"Genre\":\"Course\",\"Note\":4.6,\"AnneeSortie\":2017,\"Plateformes\":[\"Switch\"],\"Tags\":[\"course\",\"famille\",\"multijoueur local\"]},{\"Id\":\"6ac744343d2ce1ef135019cf\",\"Titre\":\"Cities: Skylines II\",\"Genre\":\"Gestion\",\"Note\":3.8,\"AnneeSortie\":2023,\"Plateformes\":[\"PC\",\"PS5\",\"Xbox Series\"],\"Tags\":[\"construction\",\"simulation\",\"solo\"]},{\"Id\":\"6ac74485e5c7e7591d722dfd\",\"Titre\":\"Stardew Valley\",\"Genre\":\"Simulation\",\"Note\":4.8,\"AnneeSortie\":2016,\"Plateformes\":[\"PC\",\"Switch\",\"PS5\",\"Mobile\"],\"Tags\":[\"pixel art\",\"d\\u00E9tente\",\"coop\\u00E9ratif\"]},{\"Id\":\"6ac74485e5c7e7591d722dfe\",\"Titre\":\"Valorant\",\"Genre\":\"FPS\",\"Note\":4.1,\"AnneeSortie\":2020,\"Plateformes\":[\"PC\"],\"T":[\"narratif\",\"tour par tour\",\"coop\\u00E9":[\"narratif\",\"tour par tour\",\"coop\\u00E9ratif\"]},{\"Id\":\"6ac74485e5c7e7591d722e02\",\"Titre\":\"Mario Kart 8 Deluxe\",\"Genre\":\"Course\",\"Note\":4.6,\"AnneeSortie\":2017,\"Plateformes\"":[\"narratif\",\"tour par tour\",\"coop\\u00E9ratif\"]},{\"Id\":\"6ac74485e5c7e7591d722e02\",\"Tit":[\"narratif\",\"tour par tour\",\"coop\\u00E9ratif\"]},{\"Id\":\"6ac74485e5c7e7591d722e02\",\"Titre\":\"Mario Kart 8 Deluxe\",\"Genre\":\"Course\",\"Note\":4.6,\"AnneeSortie\":2017,\"Plateformes\":[\"Switch\"],\"Tags\":[\"course\",\"famille\",\"multijoueur local\"]},{\"Id\":\"6ac74485e5c7e7591d722e03\",\"Titre\":\"Rocket League\",\"Genre\":\"Sport\",\"Note\":4.3,\"AnneeSortie\":2015,\"Plateformes\":[\"PC\",\"PS5\",\"Xbox Series\",\"Switch\"],\"Tags\":[\"comp\\u00E9titif\",\"voiture\",\"arcade\"]},{\"Id\":\"6ac74485e5c7e7591d722e04\",\"Titre\":\"Terraria\",\"Genre\":\"Bac \\u00E0 sable\",\"Note\":4.7,\"AnneeSortie\":2011,\"Plateformes\":[\"PC\",\"Switch\",\"PS5\",\"Mobile\"],\"Tags\":[\"exploration\",\"construction\",\"coop\\u00E9ratif\"]},{\"Id\":\"6ac74485e5c7e7591d722e05\",\"Titre\":\"Counter-Strike 2\",\"Genre\":\"FPS\",\"Note\":4.5,\"AnneeSortie\":2023,\"Plateformes\":[\"PC\"],\"Tags\":[\"comp\\u00E9titif\",\"tir\",\"\\u00E9quipe\"]},{\"Id\":\"6ac74485e5c7e7591d722e06\",\"Titre\":\"Hades II\",\"Genre\":\"Rogue-like\",\"Note\":4.7,\"AnneeSortie\":2024,\"Plateformes\":[\"PC\",\"Switch\"],\"Tags\":[\"rogue-like\",\"action\",\"solo\"]},{\"Id\":\"6ac74bb64df944b132a900c4\",\"Titre\":\"Un jeu de test\",\"Genre\":\"Puzzle\",\"Note\":3.5,\"AnneeSortie\":0,\"Plateformes\":[],\"Tags\":[]}]"
127.0.0.1:6379>
```

Le cache ne contient pas les prix.

Pendant 43 secondes l'API servira un catalogue sans prix.

```bash
DEL cache:catalogue:tous
```

## Q12

_`KEYS *` liste toutes les clés : relevez-les, et dites à quoi sert chacune. (Sur un serveur de production avec des millions de clés, `KEYS *` est à proscrire : il bloque Redis le temps de tout parcourir.)_

```bash
## Classement des joueurs trié par score (ZSET).
"classement:global"

## File d'attente pour le matchmaking (LIST).
"file:matchmaking"

## Compteur du nombre de parties jouées (STRING).
"compteur:parties"

## Score du joueur 7 (STRING).
"joueur:7"
```

## Q13

_Comparez les joueurs de PostgreSQL et ceux de Neo4j. \_Luna_ existe-t-elle dans les deux ? Que se passera-t-il si _Luna_ essaie d'acheter un jeu ?\_

```sql
SELECT * FROM "Players";
```

Luna n'est pas dans PostgreSQL, mais elle existe dans Neo4j.

Si Luna essaie d'acheter un jeu, ça va planter car il n'y a pas de contrat pour ça entre Neo4j et PostgreSQL (une erreur de sécurité).

## Q14
