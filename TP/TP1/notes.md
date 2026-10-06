# Notes TP1

**Auteur :** Louis AMEDRO, BUT3 APP

---

## Q1

*Le fichier déclare quatre services alors que le TP d'aujourd'hui n'en utilise qu'un. Pourquoi, à votre avis ?*

Le fichier est séparé en 4 services pour pouvoir lancer les conteneurs de manière indépendante les uns des autres et aussi pour pouvoir les éteindre d'un seul coup, en une seule commande.

## Q2

*Trois services ont une section `volumes`, un seul n'en a pas. Lequel, et qu'est-ce que ça implique pour ses données ?*

Les trois services (postgres, mongo et neo4j) ont des volumes définis, mais pas Redis. Cela signifie que les données de Redis ne seront pas sauvegardées et seront effacées à l'arrêt du conteneur.

## Q3

*Que signifie la ligne `"15432:5432"` ? Les deux nombres ne désignent pas la même chose.*

Le port 15432 (à gauche des deux points) est le port sur lequel PostgreSQL est accessible depuis l'hôte (ma machine). Le port 5432 (à droite des deux points) est le port sur lequel PostgreSQL est accessible depuis son conteneur.

## Q4

*Les mots de passe sont écrits en clair dans le fichier. Est-ce acceptable ici ? Le serait-ce sur un serveur de production ?*

Oui, dans le cadre d'un TP c'est acceptable ou en développement local.

Non, ce n'est pas acceptable sur un serveur de production car il est écrit en clair et peut être lu par n'importe qui. Ces mots de passes doit être écrit dans des variables d'environnement `.env`, qui lui même est dans le `.gitignore` pour ne pas être commit (partagé sur Git).

## Q5

*Les commandes ci-dessus utilisent `docker compose exec`. Qu'est-ce que ça fait exactement ? Où s'exécute la commande `redis-cli` ?*

`docker compose exec` permet d'exécuter une commande dans un conteneur déjà démarré.

La commande `redis-cli` (par exemple : `docker compose exec redis redis-cli ping`) est exécutée dans le conteneur Docker nommé `pixelhub-redis`.

## Q6

*La base de données tourne dans un conteneur Docker, et la chaîne de connexion dit `Host=localhost`. Pourquoi est-ce que ça fonctionne ? (Indice : relisez votre réponse à la Q3.)*

Ça fonctionne car le service `postgres` expose bien son port 5432 vers l'hôte au port 15432.

## Mon port/lien

https://127.0.0.1:5199

## Q7

*Arrêtez l'application (Ctrl+C), relancez-la. Les trois joueurs sont-ils dupliqués ? Pourquoi ?*

Non, les joueurs ne sont pas dupliqués, car la commande sert juste à les afficher et non à les créer.

## Q8

*Dans votre code, que vaut `player.Id` avant l'appel à `SaveChangesAsync()` ? Et après ? Qui a choisi la valeur 4 : votre code, EF Core ou PostgreSQL ?*

Avant : 0, car c'est la valeur par défaut pour un int en C# et que la valeur n'est pas encore inscrite dans la BDD.

Après : 4, car c'est la valeur que PostgreSQL a choisi.

## Q9

*Que valent les soldes après le `ROLLBACK` ? Imaginez maintenant que le serveur s'éteigne entre les deux `UPDATE`, sans transaction : dans quel état serait la base, et qui y perdrait ?*

Les soldes resteraient, car le ROLLBACK annule toutes les modifications.

Si le serveur s'éteint entre les deux UPDATE sans transaction, la base serait dans un état instable et Krayz aurait perdu 100 et Nova n'aurait rien gagné.

## Q10

*Pendant la transaction de A, quel solde de Nova voyaient B et l'API ? Pourquoi l'`UPDATE` de B a-t-il dû attendre ? Imaginez que A et B soient deux achats lancés au même instant par Nova sur deux téléphones : que pourrait-il se passer si la base ne bloquait pas B ?*

Pendant la transaction de A, B et l'API voyaient le même solde: 1200.

L'UPDATE de B a dû attendre car il y avait un verrou sur la ligne de A.

Si la base ne bloquait pas B, il y aurait eu une condition de concurrence: A et B auraient chacun retiré 100, ce qui aurait donné 1000 à chacun.

## Q11

*Qui a refusé l'achat d'Ombre : votre code C# ou la base ? Pourquoi le `SELECT` suivant a-t-il échoué lui aussi ? Après le `ROLLBACK`, la contrainte existe-t-elle toujours, et qu'est-ce que ça vous apprend sur PostgreSQL ?*

Qui a refusé l'achat d'Ombre : votre code C# ou la base ?
-> C'est la base (PostgreSQL), car la contrainte a été violée.

Pourquoi le SELECT suivant a-t-il échoué lui aussi ?
-> Dès qu'une erreur survient dans une transaction PostgreSQL, la transaction passe à l'état "avorté" (`aborted`). La base refuse alors toutes les requêtes suivantes jusqu'au `ROLLBACK`.

Après le ROLLBACK, la contrainte existe-t-elle toujours, et qu'est-ce que ça vous apprend sur PostgreSQL ?
-> Non, elle n'existe plus. Cela apprend que dans PostgreSQL, les modifications de structure sont transactionnelles et sont donc annulées par un `ROLLBACK`.

## Q12

*Que renvoie `TTL temporaire` au fil des secondes, puis une fois les 10 secondes écoulées ? Et `GET temporaire` ? Citez une donnée de PixelHub qui gagnerait à disparaître toute seule.*

`TTL temporaire` retourne un entier qui diminue de 1 toutes les secondes. Une fois les 10 secondes écoulées, le TTL expire et la clé est supprimée et retourne -2.

Citez une donnée de PixelHub qui gagnerait à disparaître toute seule.
-> Les tokens de session des utilisateurs

## Q13

*`INCR` lit, ajoute 1 et réécrit en **une seule** commande. Pourquoi est-ce plus sûr que de faire un `GET`, d'ajouter 1 dans le code C#, puis un `SET`, si 200 joueurs déclenchent le compteur au même moment ? (Pensez à ce que vous avez vu en 5.2.)*

-> `INCR` est atomique (c'est une opération indivisible), il n'y a donc pas de risque de concurrence. Contrairement à `GET`, `SET` qui ne sont pas atomiques, il y a un risque de concurrence.

Par exemple : Si deux joueurs appuient en même temps sur le bouton, les deux "GET" arrivent en même temps, les deux calculent 1+1, mais comme les deux "SET" arrivent en même temps, il ne sera incrémenté que d'une seule unité.

## Q14

*Les deux jeux sont dans la même collection, mais n'ont pas les mêmes champs (`plateformes` est une liste, `multijoueur` un objet imbriqué). Aurait-on pu ranger les deux dans une même table PostgreSQL ? À quel prix ? Rapprochez votre réponse de l'exercice du catalogue de jeux (section 2 du cours `CM1_etudiants_Panorama.md`).*

Oui, en ajoutant des colonnes pour chaque champ possible, mais cela créerait beaucoup de colonnes vides.

## Q15

*La requête « ami d'un ami » se lit presque comme un dessin. Comment l'écririez-vous en SQL, avec une table `Amities(joueur_id, ami_id)` ? Combien de jointures faudrait-il pour « ami d'un ami d'un ami » ?*

```sql
SELECT j3.pseudo
FROM "Joueurs" j1
JOIN "Amities" a1 ON j1.id = a1.joueur_id
JOIN "Amities" a2 ON a1.ami_id = a2.joueur_id
JOIN "Joueurs" j3 ON a2.ami_id = j3.id
WHERE j1.pseudo = 'Ana';
```

Pour « ami d'un ami d'un ami » (3 relations d'écart), il faudrait 3 jointures sur `Amities` et 1 jointure sur `Joueurs` pour obtenir le pseudo final.

## Q16

*Pourquoi `DELETE` seul a-t-il été refusé ? Que fait `DETACH` de plus ?*

Neo4j refuse de supprimer un noeud s'il possède encore des relations connectées.

`DETACH DELETE` supprime automatiquement toutes les relations attachées au noeud avant de supprimer le noeud lui-même.

## Q17

*La clé `survivant` a-t-elle survécu au `stop` ? Au `down` ? Et les joueurs de PostgreSQL ? Expliquez la différence avec le mot **volume**.*

La clé `survivant` a-t-elle survécu au `stop` ?
-> Oui

Au `down` ?
-> Non

Et les joueurs de PostgreSQL ?
-> Les joueurs sont revenus comme ils étaient avant le `down`.

Expliquez la différence avec le mot **volume**.
-> Permet de persister les données.
