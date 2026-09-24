# Roman Pichon Herrera — notes de développement

Site statique (HTML/CSS/JS vanilla, aucune dépendance de build) pour Roman Pichon Herrera.
Design inspiré de decimalstudios.com (`/work` et `/info`), volontairement simplifié.

## Liens utiles

- Repo GitHub : https://github.com/dbasso31/romanpichonherrera (public, branche `main`)
- Site en ligne (GitHub Pages) : https://dbasso31.github.io/romanpichonherrera/
  - Redéployé automatiquement à chaque `git push` sur `main` (~1 min)
- Instagram : https://www.instagram.com/roman.p.herrera/

## Structure

| Fichier | Rôle |
|---|---|
| `index.html` | Page "Work" — thème sombre, grille de projets |
| `info.html` | Page "Info" — thème clair, texte |
| `style.css` | Feuille de style partagée (toute la typo, couleurs, layout) |
| `info old.html` | Ancienne version d'info.html (backup, non référencée nulle part) |

Le JS est inline en bas de chaque page (pas de fichier `.js` séparé).

## Tester en local

**Ne pas ouvrir les fichiers directement en `file://`** : l'embed YouTube échoue (erreur 153) car YouTube exige une origine http(s).

```bash
cd "/Users/davidbasso/Documents/_PROD_BASSO.IO/Claude/roman" && python3 -m http.server 8000
```

Puis ouvrir `http://localhost:8000/index.html`.

**Depuis un mobile sur le même Wi-Fi** : utiliser l'IP locale du Mac (`ipconfig getifaddr en0`), ex. `http://192.168.1.149:8000/index.html`. Le pare-feu macOS peut bloquer Python.

**Cache** : si une modif CSS ne s'affiche pas (surtout Safari mobile), fermer l'onglet et le rouvrir, ou ajouter un paramètre de version (`style.css?v=2`) au `<link>`.

## Système de style

### Thèmes (variables CSS)

Les couleurs sont des variables CSS (`--bg`, `--fg`, `--muted`, `--line`, `--placeholder`) définies dans `:root` (thème sombre par défaut) et surchargées par `body[data-theme="light"]`.

- `index.html` → `<body data-theme="dark" data-page="work">`
- `info.html` → `<body data-theme="light" data-page="info">`

`data-page` sert pour d'éventuels ajustements spécifiques à une page, indépendamment du thème couleur.

### 2 styles de texte seulement

1. **Inter 14px**, `line-height: 22px`, weight 300 — défini une seule fois sur `html, body`, hérité par tout le reste (nav, paragraphes, `.client`...).
2. **"SERIF" : Times New Roman 17px**, weight 400, letter-spacing `0.015em` — défini une seule fois sur `.logo, .item h2`.

Ne pas ajouter d'autres `font-size` / `font-family` ailleurs sans raison : c'est un choix de design assumé.

### Grille de projets (`.work-grid`)

Grille CSS à **12 colonnes**, `column-gap: 40px`, `row-gap: 100px`. Chaque `.item` porte une classe qui définit sa largeur/position :

| Classe | `grid-column` | Comportement |
|---|---|---|
| `c-full` | `span 12` | pleine largeur |
| `c-wide` | `span 9` (10 sous 1024px) | large |
| `c-half` | `span 6` | moitié, auto-placé à gauche |
| `c-third` | `span 4` | tiers, auto-placé à gauche |
| `c-half-offset` | `7 / span 6` | moitié collée à droite |
| `c-third-offset` | `8 / span 4` | tiers à droite (laisse 1 colonne vide au bord) |

**Mobile (≤767px)** : on garde volontairement l'asymétrie (comme decimalstudios.com/work) au lieu de tout passer en pleine largeur : `c-full`/`c-wide` = 12 col, `c-half`/`c-third` = 9 col à gauche, `c-half-offset` = `5 / span 8`, `c-third-offset` = `6 / span 7` (collés à droite). Le `column-gap` passe à 12px en mobile — indispensable, sinon les 11 gouttières de 40px provoquent un débordement horizontal (une grille CSS réserve l'espace de toutes les colonnes même si un item les couvre toutes).

### Nav

- "Work" / "Info" : liens internes avec la classe `nav-link`, soulignement animé (`0.6s cubic-bezier(0.165, 0.84, 0.44, 1)` — repris de `.link-secondary` de decimalstudios.com).
- "Insta" : lien externe (`target="_blank"`, `rel="noopener noreferrer"`).
- Sous 400px de large, l'espacement de la nav passe de 28px à 16px.

### Transition de page

Tout le contenu est dans un `<div class="page">`. Au chargement : fade + slide-up (`opacity .8s ease-out, transform .8s ease-in-out`, repris de decimalstudios.com). Au clic sur un lien `a[href$=".html"]`, le script ajoute `.is-leaving` (fade + slide vers le haut) puis navigue après 800ms.

### Embed YouTube (facade cliquable)

Pour éviter d'afficher l'UI YouTube avant le clic : chaque vidéo est un `<button class="yt-facade" data-yt-id="...">` contenant juste une `<img>` (miniature `https://img.youtube.com/vi/ID/sddefault.jpg`). Au clic, un script remplace le bouton par un vrai `<iframe>` YouTube avec `autoplay=1`.

- Utiliser `sddefault.jpg` (et non `maxresdefault.jpg`, qui renvoie 404 pour certaines vidéos).
- Le ratio du conteneur `.thumb` doit correspondre au ratio natif de la vidéo (`--ratio: calc(16 / 9)` pour du 16:9), sinon YouTube ajoute des bandes noires.

Vidéos actuellement intégrées :
- `54fea7wuV6s` — The Blaze - Territory
- `XwxA_oOzMDY` — Ibeyi - Aset

Les autres `.item` (projets 03 à 10) sont encore des rectangles gris placeholders (`.thumb` vide) avec des titres/clients génériques.

## Décisions de design notables

- Textes de la page Info = placeholders (lorem ipsum + filler), à remplacer par du vrai contenu.
- Aucun footer (supprimé volontairement des deux pages).
- Les rectangles gris (`--placeholder`) remplacent les images du site d'origine ; ce n'est pas un bug, c'est le comportement voulu pour les projets sans média.

## Reste à faire / idées

- Remplacer les projets placeholders (03 à 10) par de vrais contenus (vidéo ou image).
- Remplacer le texte placeholder de `info.html`.
- Supprimer `info old.html` s'il ne sert plus.
- Ajouter un `.gitignore` (au moins `.DS_Store`).
- Le lien "Contact" de la nav d'origine a été remplacé par "Insta" (pas de page contact pour l'instant).
- Vérifier le rendu sur Safari iOS réel (comportement mobile de la grille, transitions).
