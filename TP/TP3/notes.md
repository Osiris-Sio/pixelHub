# Notes TP3

**Auteur :** Louis AMEDRO, BUT3 APP

---

## Q1

_`INCR` est une opération **atomique**. Qu'est-ce que ça veut dire concrètement, et pourquoi est-ce important si mille joueurs terminent une partie en même temps ? Comment auriez-vous fait la même chose en SQL, et quelle différence cela fait-il ?_

Une opération atomique signifie qu'elle est exécutée en une seule fois, sans interruption, ou pas du tout.

C'est important car si 1 000 joueurs terminent en même temps, chaque `INCR` est traité de façon séquentielle par Redis. Aucune opération ne s'entremêle, le compteur augmentera de 1 000 (en comptant les uns après les autres).

En SQL, on ferait :

```sql
UPDATE compteur_parties SET valeur = valeur + 1;
```

**Différence :** En SQL, 1 000 accès simultanés sur la même ligne créent des blocages et de la latence liée au disque. Redis réalise l'opération directement en mémoire vive (RAM), sans blocage, et peut ainsi exécuter des dizaines de milliers d'opérations par seconde.

## Q2

_On aurait pu sérialiser tout le joueur en JSON dans une simple `String`. Décrivez les étapes qu'il aurait alors fallu faire pour ajouter 250 XP, et le problème qui apparaît si deux requêtes le font en même temps._

Il faudrait :

1. Récupérer le joueur
2. Désérialiser le joueur en JSON
3. Ajouter 250 XP au joueur
4. Sérialiser le joueur en JSON
5. Mettre à jour le joueur dans Redis

Si deux requêtes lisent le joueur en même temps (ex. 1 000 XP), chacune prépare sa propre mise à jour et la dernière à écrire écrasera les modifications de l'autre (au lieu de cumuler les gains).

Avec un Hash (`HINCRBY joueur:7 xp 250`), l'opération est atomique et exécutée directement par Redis, sans transférer tout l'objet.

## Q3

_Les deux dernières commandes ne renvoient pas la même chose. Expliquez la différence, et dites laquelle vous utiliserez pour afficher un « top 3 »._

`ZREVRANGE` renvoie le top 3 dans l'ordre décroissant (du premier au troisième).

`ZRANGE` renvoie le top 3 dans l'ordre croissant (du dernier au troisième).

Pour afficher un top 3, on utiliserait `ZREVRANGE`.

## Q4

_`ZREVRANK` renvoie un nombre. Si vous affichez ce nombre tel quel au joueur, que va-t-il lire ? Que faut-il faire ?_

`ZREVRANK` retourne le rang du joueur dans le classement. (Attention, ce rang commence à partir de 0, donc il faut ajouter 1 pour l'afficher au joueur)

## Q5

_Qui sort de la file au `RPOP`, et pourquoi ? Qu'aurait donné `LPOP` à la place ?_

Avec `RPOP`, c'est le dernier joueur, Nova, entré dans la file qui sort (LIFO - Last In First Out). `LPOP` aurait fait sortir le premier joueur entré dans la file (FIFO - First In First Out), Ombre.

## Q6

_Quelles valeurs obtenez-vous ? Que renvoie `TTL` sur une clé sans expiration (testez avec `compteur:parties`) ? Que signifie chacune des valeurs de retour ?_

J'obtiens la valeur `-2`. Pour le compteur:parties, j'obtiens la valeur `-1`.
La valeur `-2` signifie que la clé existe mais n'a pas de TTL.
La valeur `-1` signifie que la clé n'existe pas.

## Q7

_Personne n'a lancé de tâche de nettoyage. Quel code auriez-vous dû écrire pour obtenir le même comportement avec PostgreSQL, et quels risques ce code introduit-il ?_

Il aurait fallu créer une tâche planifiée en arrière-plan qui est exécutée régulièrement :

```sql
DELETE FROM joueurs_en_ligne WHERE dernier_ping < NOW() - INTERVAL '30 seconds';
```

Les `DELETE` fréquents peuvent bloquer les autres requêtes et dégradent les performances (utilisation des ressource).Et si le nettoyeur plante ou s'arrête, les données s'accumulent.

## Q8

_Ce service est enregistré en **singleton**. Cherchez dans la documentation de StackExchange.Redis ce qu'est un `ConnectionMultiplexer` et ce qui se passerait si on en créait un à chaque requête HTTP. Pourquoi ce bug serait-il **invisible** aujourd'hui en TP ?._

`ConnectionMultiplexer` c'est un gestionnaire d'activités et d'états de connexion à Redis.

Si on en créait un à chaque requête HTTP, il y aurait un nombre croissant de connexions à Redis, ce qui pourrait entraîner des blocages et des ralentissements.

Il serait invisible car le serveurRedis est local et le nombre de requêtes est faible.

## Q10

_Supprimez `Order.Descending` des deux méthodes, relancez, et regardez `/leaderboard/top/5`. Le résultat est-il visiblement faux ? Pourquoi ce bug est-il particulièrement dangereux ?_

Oui, le résultat est visiblement faux, car il affiche le top 5 des moins bons joueurs. Ce bug est particulièrement dangereux car il ne génère pas d'erreur, ce qui le rend difficile à détecter.

## Q12

_Combien de lignes avez-vous modifiées dans vos **endpoints** pour ajouter ce cache ? Pourquoi ? Quel choix de la séance 2 rend cela possible ?_

Il n'y a eu aucune ligne modifiée dans les endpoints. Les endpoints (`/games`) n'ont pas bougé d'une seule ligne car ils dépendent uniquement de l'interface `IGameCatalog` injectée par le conteneur de dépendances.

Le choix de la séance 2 qui rend cela possible est l'utilisation de l'injection de dépendances avec une interface (pattern Décorateur : `CachedGameCatalog` implémente `IGameCatalog` et enveloppe `MongoGameCatalog` de manière totalement transparente pour les contrôleurs/endpoints).

## Q13

```json
// 1er appel (Cache miss - initialisation Mongo)
{
  "nombre": 21,
  "millisecondes": 172.0128
}

// Appels suivants (Cache hit - servi par Redis)
{
  "nombre": 21,
  "millisecondes": 1.5236
}

// Après 60 secondes (Cache miss - expiration du TTL)
{
  "nombre": 21,
  "millisecondes": 7.7096
}
```

- **Le gain est-il spectaculaire ?**  
  En relatif oui (on passe de 7-172 ms à 1.5 ms, soit 5 à 100 fois plus rapide). Dans l'absolu, pour un seul utilisateur sur 21 jeux en local, la différence est à peine perceptible.
- **Conditions où le cache devient décisif :**
  1. **Forte charge / concurrence :** pour protéger la base de données (MongoDB) d'une surcharge quand des milliers d'utilisateurs demandent la même page.
  2. **Requêtes lourdes :** calculs longs, tris complexes ou agrégations coûteuses.
  3. **Latence réseau :** quand la base de données principale est sur un serveur distant.

## Q14

_Un administrateur corrige le titre d'un jeu directement dans MongoDB. Combien de temps les joueurs verront-ils l'ancien titre, et pourquoi ? Proposez deux stratégies pour réduire ce délai, avec leur inconvénient._

Les joueurs verront l'ancien titre pendant au maximum 60 secondes. C'est la durée du TTL fixé sur la clé de cache dans Redis. Tant que ce délai n'est pas écoulé, l'application lit Redis sans interroger MongoDB.

**Deux stratégies pour réduire ce délai :**

1. **Raccourcir la durée du TTL** (ex. 5 ou 10 secondes) :
   - _Inconvénient :_ Plus de _cache misses_, donc plus de requêtes qui frappent MongoDB, ce qui diminue le bénéfice du cache.
2. **Invalidation active du cache (suppression de la clé)** :
   - Dès qu'un jeu est modifié via l'API, l'application supprime la clé `cache:catalogue:tous`.
   - _Inconvénient :_ Plus complexe à implémenter, et inefficace si l'administrateur modifie la donnée directement dans MongoDB (Compass, shell) sans passer par l'API.
