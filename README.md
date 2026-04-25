# ROLLIN WHEELS

Jeu de roue de fortune **multijoueur en temps réel**. Deux joueurs s'affrontent
depuis leurs téléphones via un lien d'invitation. Chaque joueur tourne une roue
qui ajoute ou soustrait des montants en FCFA. Le gagnant encaisse la différence
des deux scores auprès du perdant.

Stack : HTML / CSS / JS vanille en un seul fichier `index.html`, Firebase
Realtime Database via CDN compat v9, Google Fonts (Bebas Neue + Oxanium).
Aucune autre dépendance, aucun build.

---

## Fichiers

```
.
├── index.html      # tout le jeu (HTML + CSS + JS inline)
├── firebase.json   # config Firebase Hosting
├── .firebaserc     # projet Firebase par défaut
└── README.md
```

---

## Prérequis

- Compte Firebase + projet `rollin-wheels` (déjà configuré dans le code)
- **Realtime Database** activée en région `europe-west1`
- [Firebase CLI](https://firebase.google.com/docs/cli) installé : `npm i -g firebase-tools`

---

## Règles de sécurité Realtime Database

Dans la console Firebase → Realtime Database → onglet **Rules**, coller :

```json
{
  "rules": {
    "parties": {
      "$gameId": {
        ".read": true,
        ".write": "newData.exists()"
      }
    }
  }
}
```

> Cette règle autorise lecture publique d'une partie et écriture tant que la
> donnée n'est pas effacée. La suppression (`remove()`) déclenchée par
> "Annuler" / "REJOUER" / le nettoyage 5 min côté client passe parce que
> `newData.exists()` n'est pas évalué pour les suppressions de chemins entiers.

---

## Test local

Option 1 — direct (le plus simple) :

```
ouvrir index.html dans Chrome
```

Tout fonctionne, y compris Firebase, dès que l'origine est `file://`, `localhost`
ou un domaine autorisé.

Option 2 — via l'émulateur Firebase :

```
firebase emulators:start --only hosting
```

L'app sera disponible sur `http://localhost:5000`.

---

## Déploiement

```
firebase login
firebase deploy --only hosting
```

L'URL de production sera :

```
https://rollin-wheels.web.app
```

Pour qu'un joueur en invite un autre, il partage simplement le lien généré sur
l'écran **Invitation** (par exemple
`https://rollin-wheels.web.app/#join=X7K2P9&host=JASON`).

---

## Mode d'emploi (joueur)

1. Le **host** saisit son prénom, choisit le nombre de rounds (2 à 12), et
   appuie sur **LANCER LE DÉFI ⚡**.
2. Il copie le lien généré et l'envoie à son adversaire (WhatsApp, SMS…).
3. Le **guest** ouvre le lien, saisit son prénom et appuie sur
   **ACCEPTER LE DÉFI 🔥**.
4. Le host commence : 60 secondes pour faire tourner la roue autant de fois
   que possible. Chaque spin ajoute ou retire des FCFA.
5. À la fin du timer, c'est au tour du guest. Et ainsi de suite jusqu'à
   épuisement des rounds.
6. À la fin, l'écran de résultat affiche la différence due. Le bouton
   **REJOUER** efface la partie et ramène à l'accueil.

Pendant la partie : appuyez sur **⏸** pour mettre en pause (les deux écrans
gèlent), tapez les emojis en bas de l'écran pour chambrer en temps réel.

---

## Notes techniques

- Reconnexion automatique : `localStorage` conserve `rdf_gameId` + `rdf_role`,
  l'app reprend l'état exact au rechargement.
- Bannière de réseau : surveille `.info/connected` et affiche
  *"Reconnexion…"* en cas de perte de connexion.
- Nettoyage : 5 minutes après la fin d'une partie, le client supprime le nœud
  `parties/<gameId>` dans Firebase.
