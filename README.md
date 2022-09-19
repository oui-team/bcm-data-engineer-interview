# Objectif

L’objectif de cet exercice est de créer une application standalone qui récupère
et met en base des données de production d'une centrale électrique.

**Nous regarderons le code de la même façon que nous regarderions un code de
production.**

# Contexte

Pour cet exercice, nous avons une centrale de production d'énergie qui s'appelle
`Hawes`.

Nous avons une infrastructure de monitoring qui est en capacité de remonter des
informations basiques de production d’électricité de la centrale. Ici, nous nous
intéressons uniquement à la puissance de production sur un pas de temps donné.
**Le pas de temps de mesure est toujours constant et vaut de 15 minutes**:

# Centrale Hawes

L'API permet la récupération des données enregistrées sur le site et prend deux
paramètres: `from` et `to`. `from` correspond au point de départ de la remontée
d’informations, `to` étant le point final.

L'API renvoie l’ensemble des données existantes comprises entre `from` et `to`
sous forme de segment d’une durée fixe (15 min). Toutes les données temporelles
sont renvoyées sous forme de _timestamp_. L'API de récupération de la donnée est
décrite ci-dessous:

- URL: https://interview.beta.bcmenergy.fr/hawes
- Query Params:

| Parameter | Description                   | Format       |
| --------- | ----------------------------- | ------------ |
| `from`    | The start date for the search | `DD-MM-YYYY` |
| `to`      | The end date for the search   | `DD-MM-YYYY` |

**Exemple de requête et réponse**

```bash
$> curl https://interview.beta.bcmenergy.fr/hawes?from=01-01-2020&to=05-01-2020

[
   {
      "start":1578006000,
      "end":1578006900,
      "power":710
   },
   {
      "start":1578006900,
      "end":1578007800,
      "power":627
   },
   {
      "start":1578007800,
      "end":1578008700,
      "power":518
   },
   {
      "start":1578008700,
      "end":1578009600,
      "power":750
   },
   ...
]
```

## Contraintes

Il arrive fréquemment que les APIs de centrales aient des trous de données. Il
n’existe jamais deux trous consécutifs. Dans le cas où un trou existe à la
position `N` dans le retour d’une API, on utilise la moyenne des valeurs `N - 1`
et `N + 1`. La première et la dernière valeur ne sont jamais manquantes. Cette
interpolation doit se faire sur chaque retour unitaire de l’API. Un trou de
données est facilement identifiable car l'objet correspondant au pas de temps
concerné n'est pas présent dans la réponse.

Il est à noter que dans le cadre de notre test, l'API de la centrale renvoie des
données **totalement aléatoires**.

De plus, il n’est pas rare que l'API renvoie des erreurs 500.

# Programme attendu

Comme décrit plus haut, nous attendons un programme qui soit en mesure de
pouvoir récupérer la donnée brute, la nettoyer (cf.
[Paragraphe "Contraintes"](#contraintes)) et l'insérer en base de données.

Nous ne voulons pas d’API, mais un programme standalone dans la stack de votre
choix (langage et base de données). Le nombre de frameworks utilisés doit être
tenu au strict minimum. Le choix du schéma de la ou des tables est laissé libre.

Notre programme prend 2 paramètres en entrée:

- `from` (date au format `DD-MM-YYYY`)
- `to` (date au format `DD-MM-YYYY`)

Où `from` correspond au point de départ de la remontée d’infos et `to` le point
final.

Nous attendrons également les scripts d'initialisation de la base de données
ainsi que les instructions pour la monter (docker-compose, etc).

Une brève description de la base est souhaitable pour qu'un utilisateur puisse facilement consulter les données (en se connectant directement à la base).

# Points d’attention

Comme mentionné en début d’énoncé, nous regarderons le code comme s'il avait été
écrit pour la production. Nous devons être en mesure de lancer facilement le
programme.

# Bonus

Écrire un docker-compose pour pouvoir lancer facilement l’application et la base
de données.
