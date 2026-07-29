# La Boucle — compagnon de run

Une page unique (`index.html`) pour un one-shot rogue-lite à 2 joueurs + MJ.
Trois sièges : **Cartes**, **Dés**, **MJ**. Le MJ déclenche les offres, chaque
joueur pioche 3 propositions parmi ce qui lui reste d'éligible, en choisit une.

Aucune dépendance à installer. Tout tient dans un fichier.

---

## 1. Mettre en ligne sur GitHub Pages

1. Créer un dépôt **public** sur GitHub.
2. Y déposer `index.html` à la racine (bouton *Add file → Upload files*).
3. *Settings → Pages → Source : Deploy from a branch*, branche `main`, dossier `/ (root)`, **Save**.
4. Au bout d'une minute, l'adresse est `https://TON-PSEUDO.github.io/NOM-DU-DEPOT/`.

C'est ce lien que tu envoies aux joueurs. Ils l'ouvrent sur leur téléphone.

---

## 2. Brancher Firebase (pour que les trois téléphones se parlent)

Sans cette étape l'app tourne en **mode local** : elle marche, mais chaque
appareil a sa propre partie. Utile pour tester seul dans plusieurs onglets ;
inutilisable à table.

1. [console.firebase.google.com](https://console.firebase.google.com) → **Ajouter un projet**. Tu peux désactiver Google Analytics.
2. Menu de gauche → **Realtime Database** → *Créer une base de données*.
   Choisis la région **europe-west1**, puis *Démarrer en mode test*.
3. Roue crantée → **Paramètres du projet** → section *Vos applications* →
   icône **`</>`** (Web) → enregistre l'app → copie le bloc `firebaseConfig`.
4. Ouvre `index.html`, cherche `FIREBASE_CONFIG` (vers la ligne 320) et colle les valeurs.

> **Piège fréquent :** Firebase omet parfois `databaseURL` du bloc copié.
> Reprends-la sur la page Realtime Database, en haut. Elle ressemble à
> `https://mon-projet-default-rtdb.europe-west1.firebasedatabase.app`.
> Tant que ce champ est vide, l'app reste en mode local.

5. Onglet **Règles** de la Realtime Database, remplace par :

```json
{
  "rules": {
    "sessions": {
      "$code": { ".read": true, ".write": true }
    }
  }
}
```

Le mode test expire au bout de 30 jours ; ces règles-là n'expirent pas.
Elles laissent en écriture tout ce qui est sous `/sessions` et rien d'autre.
Pour une table de trois personnes c'est suffisant, mais n'importe qui
connaissant l'adresse **et** le code de table peut écrire : prends un code
un peu long (`BOUCLE7X2`) plutôt que `TEST`.

Quota gratuit Firebase : 1 Go stockés, 10 Go/mois de trafic. Une partie
consomme quelques kilo-octets.

---

## 3. À table

- Tout le monde ouvre le lien et tape **le même code de table**.
- Chaque joueur prend son siège ; un siège occupé apparaît *PRIS*.
- Le MJ prend le siège MJ, puis pilote :
  - **Proposer une aptitude aux deux** → chaque joueur reçoit son propre tirage de 3.
  - **Proposer le choix de spécialité** → les deux branches, une seule fois.
  - **Nouvelle run** → remet à zéro les jetons *1× par run*, garde les aptitudes.
  - **Annuler le dernier choix** → rend l'aptitude et rouvre exactement la même offre.
  - Par joueur : relancer le tirage, retirer l'offre, ajouter/retirer une aptitude à la main, forcer une spécialité.
- Les joueurs peuvent **passer** une offre d'aptitude (pas une offre de spécialité).
- Chaque aptitude *1× par run* possède un jeton que le joueur coche quand il l'utilise.

---

## 4. Modifier le contenu

Tout est dans `index.html`, dans les deux blocs commentés en haut du script :

| Bloc | Ce qu'il contient |
|---|---|
| `1 ▸ CONFIG` | clés Firebase, `OFFER_SIZE` (3 cartes par offre), `REVEAL_LOCKED` |
| `2 ▸ DONNÉES` | tous les noms, textes, branches |

Une aptitude s'écrit :

```js
{id:"mulligan", name:"Mulligan", text:"Remettre jusqu'à deux cartes…", charge:true}
```

`id` doit être unique et ne plus jamais changer (il est stocké dans les parties
en cours). `charge:true` ajoute le jeton *1× par run*.

`REVEAL_LOCKED = false` masque les aptitudes non acquises derrière des `???`
dans l'arbre du joueur, pour garder la découverte. Passe-le à `true` si tu
préfères qu'ils voient l'arbre complet dès le départ. Le MJ voit toujours tout.

---

## 5. Tester seul

Ouvre `index.html` en local et lance trois onglets sur le même code : le mode
local synchronise les onglets d'un même navigateur. Si la page reste blanche
en double-cliquant le fichier, sers-le avec `python3 -m http.server 8000` puis
va sur `http://localhost:8000`.
