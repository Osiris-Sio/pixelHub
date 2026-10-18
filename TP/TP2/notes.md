# Notes TP2

**Auteur :** Louis AMEDRO, BUT3 APP

---

## Q1

*Vous retrouvez le champ `_id`, déjà aperçu au TP 1 (étape 6.2), que personne n'a écrit dans `games.json`. Cette fois, intéressez-vous à **qui l'a fabriqué** : le serveur MongoDB, ou l'outil qui a envoyé les documents (`mongoimport`) ? Comparez avec l'`Id` 4 de *Pixel* au TP 1 (Q8) : qui l'avait choisi, et à quel moment le connaissiez-vous ?*

L'id a été fabriqué par mongoimport car c'est lui qui a envoyé les documents.

Dans le TP1, c'était le serveur Postgre qui fabriquait l'id.
On connaissait l'id uniquement après qu'il est inséré dans la base de données. En revanche, avec MongoDB, on peut savoir l'id avant même qu'il soit inséré dans la base de données.

## Q2

_À l'exercice 5, `plateformes` est un tableau et vous avez écrit `{ plateformes: "Switch" }`, sans opérateur particulier. Comment auriez-vous fait la même chose en SQL, avec un modèle relationnel normalisé ?_

Il faudrait rechercher avec des jointures entre la table jeux et la table plateformes.

## Q3

_Vous tapez `--authenticationDatabase admin` depuis le TP 1 sans qu'on vous ait dit pourquoi. Pourquoi faut-il l'ajouter aux commandes `mongoimport` et `mongosh` ? Où se trouve le compte `pixelhub` ?_

On ajoute `--authenticationDatabase admin` pour pouvoir se connecter au compte pixelhub avec son "badge" administrateur (`admin`) pour pouvoir accéder à la base de données `pixelhub` et par exemple, y importer les documents.

Donc, en effet, le compte pixelhub se trouve dans la base de données `admin`.

## Q4

_Personne ne vous a arrêté. Est-ce une bonne nouvelle ou un problème ? Argumentez en imaginant un projet à quatre développeurs, six mois plus tard._

C'est un problème car n'importe qui peut se ajouter n'importe quoi dans la base de données.

Chaque développeur peut donc mettre n'importe quoi dans la base de données, ce qui pourrait causer des problèmes plus tard (exemple : note vs la_note).

## Q5

_Le résultat de l'exercice 9 vous surprend-il ? Que s'est-il passé exactement, et pourquoi n'y a-t-il eu **aucun message d'erreur** ?_

Non, ce n'est pas surprenant. La collection a été créée avec une minuscule (jeux) et non avec une majuscule (Jeux). En effet, MongoDB n'a pas de schéma, donc il ne va pas vérifier que le nom de la collection existe bel et bien.

Il affiche seulement un message vide pour dire qu'il n'y a pas de collection qui correspond à la requête, car MongoDB est sensible à la casse, et considère la collection `Jeux` comme vide, même si cette collection n'existe pas.

## Q6

_L'exercice 16 échoue. Recopiez le message d'erreur, et expliquez pourquoi MongoDB refuse. Quelle commande utiliseriez-vous si vous vouliez **réellement** remplacer le document en entier ?_

```cmd
MongoInvalidArgumentError: Update document requires atomic operators
```

MongoDB refuse car il faut utiliser des opérateurs atomiques pour modifier un document.

Pour remplacer un document en entier, il faut utiliser la commande `replaceOne`.

```js
db.jeux.replaceOne({ titre: 'Valorant' }, { note: 4.4 });
```

## Q7

_À l'exercice 12, vous avez ajouté un champ `nbVotes` à un seul document, alors que les dix autres n'en ont pas. Quelle commande SQL aurait été nécessaire pour faire l'équivalent en relationnel, et quelle en aurait été la conséquence sur les autres lignes ?_

La commande SQL aurait été d'ajouter une colonne `nbVotes` à la table `jeux`, avec que toutes les lignes de la table `jeux` auraient eu un champ `nbVotes = NULL`. (ou avec un 0 si on met `DEFAULT 0`)

## Q8

_Lisez `IGameCatalog`. Y a-t-il un seul endroit dans cette interface où le mot « Mongo » apparaît ? Pourquoi est-ce important, à votre avis ?_

"Mongo" n'apparaît nulle part dans l'interface `IGameCatalog`. C'est important car l'interface est générique et ne dépend pas de la technologie utilisée (ici, MongoDB). Ainsi, on peut changer de base de données sans changer l'interface (flexibilité).

## Q9

_Quelle exception obtenez-vous, et sur quel champ ? Expliquez la cause : qu'est-ce que le driver a essayé de faire, et pourquoi n'a-t-il pas su ?_

```cmd
fail: Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware[1]
      An unhandled exception has occurred while executing the request.
      System.FormatException: Element 'nombreArmes' does not match any field or property of class PixelHub.Api.Models.Game.
```

J'obtiens une exception `System.FormatException` sur le champ `nombreArmes`.

Le driver a essayé de mapper un objet de type `Game` avec les données de la collection `jeux`, mais le document `Hades II` a le champ `nombreArmes`. Or, le `Game` n'a pas ce champ, le driver a donc généré une exception.

## Q10

_`[BsonIgnoreExtraElements]` fait disparaître l'erreur, mais **au prix de quoi** ? Que devient `joueursParEquipe` quand vous appelez `/games` ?_

Certes, l'erreur disparaît, mais le champ `joueursParEquipe` (ou encore `nombreArmes`) disparaît également, car il n'a pas été mappé sur un champ de la classe `Game`.

## Q11

_Comparez les trois approches possibles pour gérer un schéma variable en C# : `[BsonIgnoreExtraElements]`, `[BsonExtraElements]`, et une hiérarchie de classes (`FpsGame : Game`, etc.). Quel est l'avantage et l'inconvénient de chacune ?_

1. `[BsonIgnoreExtraElements]` :
   - **Avantage :** Très simple et rapide à mettre en place (une seule annotation). L'application ne plante plus face aux documents ayant des champs imprévus.
   - **Inconvénient :** Perte de données côté applicatif : tous les champs non déclarés dans la classe `Game` sont ignorés et deviennent invisibles pour l'API.

2. `[BsonExtraElements]` :
   - **Avantage :** Aucune perte d'information. Tous les champs spécifiques non déclarés sont capturés dans un dictionnaire/BsonDocument et restent exploitables.
   - **Inconvénient :** On perd le typage fort de C# et l'autocomplétion pour ces champs. De plus, la sérialisation vers l'API est plus complexe (nécessite une conversion pour le JSON ASP.NET Core).

3. `Hiérarchie de classes` :
   - **Avantage :** Conserve le typage fort de C#, l'autocomplétion dans l'IDE et la validation à la compilation pour chaque variante de jeu.
   - **Inconvénient :** Approche rigide : elle oblige à créer une classe pour chaque type de jeu et à connaître les schémas à l'avance, ce qui réduit la souplesse naturelle de MongoDB.

## Q12

_Écrivez la requête SQL qui produirait le même résultat sur une table `jeux(titre, genre, note)`. Comparez les deux. Est-ce ici que MongoDB apporte quelque chose par rapport à PostgreSQL ? Si non, où l'apport se situe-t-il dans ce TP ?_

```sql
SELECT genre, AVG(note) AS note_moyenne, COUNT(*) AS nb_jeux
FROM jeux
GROUP BY genre
ORDER BY note_moyenne DESC;
```

Non, pour ce type de calcul, SQL reste plus court et naturel.

L'avantage de MongoDB est sa flexibilité, ce qui permet de stocker des champs spécifiques et variables selon les jeux, sans avoir à modifier la structure d'une table SQL.

De plus, les données imbriquées permettent de stocker directement des listes (les plateformes, les avis) au sein d'un même document sans devoir créer de tables et de jointures, ce qui rend le code plus simple à écrire et à comprendre.

## Q13

_Les requêtes d et e ne renvoient pas la même chose. Laquelle a raison ? Que demande exactement la requête d, et pourquoi trouve-t-elle *Stardew Valley* alors que Krayz lui a mis 4 ?_

C'est la requête e qui a raison.

La requête d cherche les jeux qui possèdent au moins un avis dont l'auteur est "Krayz", ET au moins un avis ayant la note 5, (aucun lien entre l'auteur et la note). Donc, dans Stardew Valley, il y a bien un avis rédigé par "Krayz" (note 4) et il y a bien un avis avec la note 5 (celui de "Nova"). Comme les deux conditions sont vraies, le jeu est sélectionné et retourné.

La requête e, utilise l'opérateur `$elemMatch` et vérifie que les deux critères (`auteur: "Krayz"` ET `note: 5`) sont vrais sur un même avis.

## Q14

_Combien d'octets coûte un avis, en moyenne ? Un document MongoDB ne peut pas dépasser 16 Mo (16 777 216 octets) : à partir de combien d'avis environ *Stardew Valley* ne pourrait-il plus en accueillir ? Est-ce un risque réel pour un jeu populaire ? Rapprochez votre réponse des critères du cours._

**Avant :**

```MongoDB
[
  { titre: 'Stardew Valley', octets: 273 },
  { titre: 'Stardew Valley', octets: 273 }
]
```

**Après :**

```MongoDB
[
  { titre: 'Stardew Valley', octets: 516 },
  { titre: 'Stardew Valley', octets: 273 }
]
```

**Calculs :**
(16 777 216 - 273) / 81 ~= 207 122 avis (environ 200 000 avis) avant saturation des 16 Mo.

C'est un risque réel pour un jeu populaire, car un jeu très populaire peut avoir beaucoup plus de 200 000 avis.

Les avis ont une croissance "non bornée". Selon le cours, une donnée "non bornée" ne doit pas être imbriquée sans limite dans un document. Il est préférable de les référencer dans une collection dédiée `avis`.
