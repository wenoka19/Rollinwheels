# ROLLIN WHEELS — Spécification Technique

## Vue d'ensemble

Jeu de roue de fortune multijoueur en temps réel. Deux joueurs s'affrontent depuis leurs téléphones (iPhone ou Android) via un lien d'invitation. Chaque joueur tourne une roue qui ajoute ou soustrait des montants en FCFA à son score. Le gagnant encaisse la différence des deux scores auprès du perdant.

---

## Stack technique

- **Frontend** : HTML / CSS / JavaScript vanille — fichier unique `index.html`
- **Temps réel** : Firebase Realtime Database via CDN compat v9
- **Polices** : Google Fonts — `Bebas Neue` + `Oxanium`
- **Aucune autre dépendance**

### CDN Firebase

```html
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
```

### Config Firebase

```js
const firebaseConfig = {
  apiKey: "AIzaSyAMEg0vSU14GbbDUvKuZcQlMNqtyFTAeUw",
  authDomain: "rollin-wheels.firebaseapp.com",
  databaseURL: "https://rollin-wheels-default-rtdb.europe-west1.firebasedatabase.app",
  projectId: "rollin-wheels",
  storageBucket: "rollin-wheels.firebasestorage.app",
  messagingSenderId: "371290056997",
  appId: "1:371290056997:web:1271b0e22a0a38fc4c52cc"
};
```

### Règles de sécurité Firebase

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

---

## Structure de données Firebase

```json
{
  "parties": {
    "ABC123": {
      "host": { "name": "JASON", "score": 45000 },
      "guest": { "name": "ABDOUL", "score": -10000 },
      "totalRounds": 4,
      "currentRound": 2,
      "roundOwner": "host",
      "timeLeft": 38,
      "phase": "waiting",
      "paused": false,
      "pausedBy": null,
      "lastEmoji": { "from": "guest", "emoji": "😂", "ts": 1714000000000 },
      "createdAt": 1714000000000
    }
  }
}
```

Valeurs possibles de `phase` : `waiting` | `playing` | `ended`

---

## Fichiers à produire

- `index.html` — tout le jeu (HTML + CSS + JS inline)
- `firebase.json` — config Firebase Hosting
- `.firebaserc` — config projet Firebase
- `README.md` — instructions de déploiement

---

## Les 5 écrans

### Écran 1 — HOME

- Logo **ROLLIN WHEELS** en `Bebas Neue`, grand, dégradé or (`#FFD100` → `#FF8C00` → `#FF5500`)
- Champ texte : prénom du joueur (max 16 caractères, majuscules)
- Sélecteur de rounds : `2 / 4 / 6 / 8 / 10 / 12` — défaut `4`
- Bouton **"LANCER LE DÉFI ⚡"** (dégradé or, texte noir)
- Action au clic :
  1. Générer un `gameId` de 6 caractères alphanumériques aléatoires (ex: `X7K2P9`)
  2. Écrire la partie dans Firebase avec `phase: "waiting"`, scores à 0, `roundOwner: "host"`
  3. Sauvegarder `gameId` et `role: "host"` dans `localStorage`
  4. Afficher l'écran Invitation

---

### Écran 2 — INVITATION

- Titre : *"Défi créé !"*
- Lien complet à partager :
  ```
  https://[window.location.origin][window.location.pathname]#join=ABC123&host=JASON
  ```
- Bouton **"Copier le lien"** → copie dans le presse-papier → affiche *"✓ Copié !"* 2 secondes
- Point vert pulsant + texte *"En attente de ton adversaire…"*
- Bouton *"Annuler"* → supprime la partie Firebase + efface localStorage + retour Home
- Listener Firebase sur `parties/gameId/guest/name` → dès qu'une valeur apparaît → basculer vers l'écran Jeu

---

### Écran 3 — JOIN

- Détecter au chargement si `window.location.hash` contient `#join=...`
- Si oui → afficher cet écran directement
- Badge doré : *"Tu es défié(e) par [NOM DU HOST]"*
- Champ texte : prénom du guest (max 16 caractères)
- Bouton **"ACCEPTER LE DÉFI 🔥"**
- Action au clic :
  1. Écrire `guest.name` dans Firebase
  2. Écrire `phase: "playing"` dans Firebase
  3. Sauvegarder `gameId` et `role: "guest"` dans `localStorage`
  4. Basculer vers l'écran Jeu
- Si la partie n'existe pas → *"Cette partie n'existe plus"* + bouton retour Home
- Si `guest.name` déjà rempli → *"Cette partie est déjà pleine"*

---

### Écran 4 — JEU

#### Layout mobile (flex colonne, centré, padding-bottom 90px)

```
┌─────────────────────────────────────┐
│  [NOM J1]      VS      [NOM J2]     │
│  [SCORE J1]          [SCORE J2]     │
│  [TIMER J1]          [TIMER J2]     │
├─────────────────────────────────────┤
│  Round X/Y  [● ● ○ ○]          [⏸] │
├─────────────────────────────────────┤
│    ⚡ TON TOUR — LANCE LA ROUE !    │
├─────────────────────────────────────┤
│              [ ROUE ]               │
├─────────────────────────────────────┤
│          +50 000 FCFA               │
├─────────────────────────────────────┤
│ 😂 🔥 💀 😭 🤣 😈 🤡 👀 💸 🙏 🤑 😤 🫵 🤦 │
└─────────────────────────────────────┘
```

#### Scoreboard

- Prénom au-dessus de chaque score box (`Bebas Neue`)
- Score box : fond `rgba(255,255,255,0.04)`, bordure `rgba(255,255,255,0.08)`, border-radius 14px
  - Joueur actif → bordure or `rgba(255,209,0,0.55)` + glow
  - Adversaire → bordure bleue `rgba(41,121,255,0.4)`, prénom grisé
- Score en `Bebas Neue` `clamp(1.6rem, 6vw, 2.2rem)`, couleur or
  - Gain : flash vert + animation scale up
  - Perte : flash rouge + animation shake
- Timer **en dessous du score** dans la même box
  - Format : `1:00` → `0:00`
  - `Bebas Neue` `2rem`, couleur or → rouge + clignotant sous 10 secondes
  - Visible uniquement pour le joueur dont c'est le tour (`opacity: 0` pour l'autre)

#### Round bar

- Texte *"Round X / Y"* à gauche
- Dots : remplis or pour les rounds passés, pulsant pour le round actuel, gris pour les futurs
- Bouton ⏸ à droite (36×36px)

#### Bannière de tour

- Tour du joueur local → *"⚡ TON TOUR — LANCE LA ROUE !"* couleur or
- Tour adversaire → *"⏳ TOUR DE [NOM]…"* couleur bleue, opacité 0.75

#### La Roue — Canvas 296×296px

**Dessin des segments :**
- 16 segments avec `createLinearGradient` centre → bord
- Positifs (verts) : `#0d6e35` → `#0f8040`
- +50K / +100K (or) : `#8a6e00` → `#b08c00`
- Négatifs (rouges) : `#8b0a2a` → `#a80d32`
- -50K / -100K (rouge vif) : `#5e0015` → `#75001b`
- Séparateurs : `rgba(0,0,0,0.4)` lineWidth 1.5
- Lueur bord arc : `rgba(255,255,255,0.07)`
- Texte : `Oxanium 800` 9-11px, blanc, ombre noire, aligné à droite
- Anneau extérieur : stroke dégradé or 4px

**Éléments autour de la roue :**
- Anneau de tick marks : `repeating-conic-gradient` masqué avec un anneau, 16 marques dorées
- Pointeur triangulaire en haut : triangle blanc/or, `filter: drop-shadow` or, animation bob (translateY -4px, 2s infinite)
- Hub cliquable (50px) : aspect métal brossé, pastille brillante au centre

**Physique du spin :**
```js
wVel = 0.26 + Math.random() * 0.38; // lancer
wAngle += wVel;
wVel *= 0.983; // décélération par frame
// stop quand wVel < 0.003
```

**Détection du segment :**
```js
function getTopSegment(angle) {
  const arc = (Math.PI * 2) / 16;
  let a = ((-Math.PI / 2) - angle) % (Math.PI * 2);
  if (a < 0) a += Math.PI * 2;
  return SEGMENTS[Math.floor(a / arc) % 16];
}
```

**Effets visuels pendant le spin :**
- `box-shadow` sur canvas : `0 0 0 3px rgba(255,209,0,0.55), 0 0 40px rgba(255,209,0,0.25)`
- Aura derrière la roue : radial-gradient or, animation scale pulse

#### Les 16 segments

```js
const SEGMENTS = [
  { label: '+10 000',  value:   10000 },
  { label: '-5 000',   value:   -5000 },
  { label: '+50 000',  value:   50000 },
  { label: '-15 000',  value:  -15000 },
  { label: '+1 000',   value:    1000 },
  { label: '-100K',    value: -100000 },
  { label: '+25 000',  value:   25000 },
  { label: '-2 000',   value:   -2000 },
  { label: '+5 000',   value:    5000 },
  { label: '-50 000',  value:  -50000 },
  { label: '+100K',    value:  100000 },
  { label: '-1 000',   value:   -1000 },
  { label: '+20 000',  value:   20000 },
  { label: '-10 000',  value:  -10000 },
  { label: '+500',     value:     500 },
  { label: '-30 000',  value:  -30000 },
];
```

#### Mécanique de jeu

1. Round 1 : `roundOwner = "host"` → le host peut spinner pendant 60 secondes
2. À chaque spin : mettre à jour le score dans Firebase immédiatement
3. Timer : le joueur actif décrémente `timeLeft` dans Firebase toutes les secondes
4. Quand `timeLeft === 0` : écrire `currentRound + 1`, basculer `roundOwner`, remettre `timeLeft: 60`
5. Si `currentRound > totalRounds` → écrire `phase: "ended"`
6. Le joueur passif écoute Firebase uniquement — il ne gère jamais le timer lui-même

**Conditions pour spinner :**
`roundOwner === monRole` ET `phase === "playing"` ET `paused === false` ET `timeLeft > 0`

#### Pause

- N'importe quel joueur peut appuyer sur ⏸
- Écrire dans Firebase : `paused: true, pausedBy: monRole`
- Les deux joueurs détectent via listener → `clearInterval` sur le timer
- Overlay :
  - Icône ⏸ animée
  - *"PAUSE"* en grand
  - *"[NOM] a mis le jeu en pause — Timer gelé ⏱"*
  - Bouton **"▶ REPRENDRE"** → écrire `paused: false, pausedBy: null`
- Emojis toujours disponibles pendant la pause

---

### Écran 5 — RÉSULTAT

- Icône 🥳 animée (rotation gauche-droite)
- Message en `Bebas Neue` :
  ```
  Félicitations [WINNER] vous avez gagné [MONTANT] FCFA auprès de [LOSER] 🥳
  ```
  - `[WINNER]` → `#FFD100`
  - `[MONTANT]` → `#00E676`, formaté `toLocaleString('fr-FR')`
  - `[LOSER]` → `#FF1744`
- Égalité exacte → *"MATCH NUL ! Personne ne doit rien 🤝"*
- Tableau 2 colonnes : scores finaux des deux joueurs (carte gagnant en or)
- Bouton **"REJOUER"** → efface localStorage + supprime partie Firebase + retour Home
- Confettis : 100 particules multicolores (or, orange, vert, bleu, rouge, blanc)

---

## Design

### Palette

| Élément | Valeur |
|---|---|
| Fond | `#000000` pur noir |
| Or principal | `#FFD100` |
| Or secondaire | `#FF8C00` |
| Vert gain | `#00E676` |
| Rouge perte | `#FF1744` |
| Bleu adversaire | `#2979FF` |
| Panel | `rgba(255,255,255,0.04)` |
| Bordure | `rgba(255,255,255,0.08)` |
| Texte atténué | `rgba(255,255,255,0.28)` |

### Typographie

- **Bebas Neue** : scores, timers, prénoms, montants, titres, boutons principaux
- **Oxanium 700** : labels, sous-titres, textes informatifs

### Animations CSS

```css
@keyframes scoreUp    { 0%,100%{transform:scale(1)} 50%{transform:scale(1.28)} }
@keyframes scoreDn    { 0%,100%{transform:translateX(0)} 30%{transform:translateX(-5px)} 70%{transform:translateX(5px)} }
@keyframes timerBlink { 50%{ opacity:0.35; } }
@keyframes ptrBob     { 0%,100%{transform:translateY(0)} 50%{transform:translateY(-4px)} }
@keyframes dotPulse   { 50%{ transform:scale(1.5); } }
@keyframes flyUp      { 0%{transform:translateY(0) rotate(-12deg) scale(1);opacity:1} 50%{transform:translateY(-170px) rotate(12deg) scale(1.5);opacity:1} 100%{transform:translateY(-300px) rotate(-6deg) scale(0.7);opacity:0} }
@keyframes toastIn    { from{opacity:0;transform:translateX(-50%) scale(0.5)} to{opacity:1;transform:translateX(-50%) scale(1)} }
@keyframes toastOut   { to{opacity:0;transform:translateX(-50%) translateY(-12px)} }
@keyframes confettiFall { to{ transform:translateY(110vh) rotate(960deg); opacity:0; } }
@keyframes resultBounce { from{transform:rotate(-10deg) scale(1)} to{transform:rotate(10deg) scale(1.12)} }
@keyframes cardIn     { from{transform:translateY(18px);opacity:0} to{opacity:1} }
@keyframes glowPulse  { from{transform:translate(-50%,-50%) scale(1)} to{transform:translate(-50%,-50%) scale(1.14)} }
```

---

## Barre d'emojis temps réel

**14 emojis :** `😂 🔥 💀 😭 🤣 😈 🤡 👀 💸 🙏 🤑 😤 🫵 🤦`

**Position :** `fixed bottom:0`, fond dégradé noir vers transparent, padding-bottom 14px

**Envoi Firebase :**
```js
db.ref('parties/' + gameId + '/lastEmoji').set({
  from: monRole, emoji: e, ts: Date.now()
});
```

**Réception Firebase :**
```js
db.ref('parties/' + gameId + '/lastEmoji').on('value', snap => {
  const data = snap.val();
  if (!data) return;
  if (data.from === monRole) return;
  if (data.ts <= sessionStartTs) return; // ignorer les anciens
  spawnFlyingEmoji(data.emoji);
  showToast(data.emoji, nomAdversaire);
});
```

---

## Reconnexion

```js
// Au chargement
const hash = parseHash();
if (hash.join) {
  showJoinScreen(hash.join, hash.host);
} else {
  const savedGameId = localStorage.getItem('rdf_gameId');
  const savedRole   = localStorage.getItem('rdf_role');
  if (savedGameId && savedRole) {
    db.ref('parties/' + savedGameId).once('value', snap => {
      if (snap.exists() && snap.val().phase !== 'ended') {
        resumeGame(savedGameId, savedRole, snap.val());
      } else {
        localStorage.clear();
        showHomeScreen();
      }
    });
  } else {
    showHomeScreen();
  }
}
```

**Indicateur réseau :**
```js
db.ref('.info/connected').on('value', snap => {
  snap.val() === false ? showReconnectingBanner() : hideReconnectingBanner();
});
```

---

## Nettoyage Firebase

```js
// Appeler quand phase === "ended"
setTimeout(() => {
  db.ref('parties/' + gameId).remove();
  localStorage.removeItem('rdf_gameId');
  localStorage.removeItem('rdf_role');
}, 5 * 60 * 1000);
```

---

## firebase.json

```json
{
  "hosting": {
    "public": ".",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [{ "source": "**", "destination": "/index.html" }]
  }
}
```

## .firebaserc

```json
{
  "projects": {
    "default": "rollin-wheels"
  }
}
```
