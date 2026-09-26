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
| `index.html` | Page "Work" — thème sombre, grille de 18 projets vidéo, "31" centré en bas |
| `info.html` | Page "Info" — thème clair, texte + image à droite, animation stabilo |
| `style.css` | Feuille de style partagée (toute la typo, couleurs, layout) |
| `img/` | Vignettes locales de certaines vidéos (`*.jpg`) et image de la page Info (`info.jpg`) |
| `info old.html` | Ancienne version d'info.html (backup, non référencée nulle part) |

Le JS est inline en bas de chaque page (pas de fichier `.js` séparé).

## Tester en local

**Ne pas ouvrir les fichiers directement en `file://`** : l'embed YouTube échoue (erreur 153) car YouTube exige une origine http(s).

```bash
cd "/Users/davidbasso/Documents/_PROD_BASSO.IO/Claude/roman" && python3 -m http.server 8000
```

Puis ouvrir `http://localhost:8000/index.html`.

**Depuis un mobile sur le même Wi-Fi** : utiliser l'IP locale du Mac (`ipconfig getifaddr en0`), ex. `http://192.168.1.149:8000/index.html`. Le pare-feu macOS peut bloquer Python.

**Cache** : si une modif CSS ne s'affiche pas (surtout Safari mobile), fermer l'onglet et le rouvrir, ou incrémenter le paramètre de version du `<link>` (actuellement `style.css?v=5` dans les deux pages — à incrémenter dans `index.html` **et** `info.html`).

## Système de style

### Thèmes (variables CSS)

Les couleurs sont des variables CSS (`--bg`, `--fg`, `--muted`, `--line`, `--placeholder`) définies dans `:root` (thème sombre par défaut) et surchargées par `body[data-theme="light"]`.

- `index.html` → `<body data-theme="dark" data-page="work">`
- `info.html` → `<body data-theme="light" data-page="info">`

`data-page` sert pour d'éventuels ajustements spécifiques à une page, indépendamment du thème couleur.

### 2 styles de texte seulement

1. **Inter 14px**, `line-height: 22px`, weight 300 — défini une seule fois sur `html, body`, hérité par tout le reste (nav, paragraphes, `.client`...).
2. **"SERIF" : Times New Roman 17px**, weight 400, letter-spacing `0.015em` — défini une seule fois sur `.logo, .item h2, .signature`.

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

### Signature "31" (index.html)

`<p class="signature">31</p>` sous la grille : style SERIF, blanc, centré (`text-align: center`), 100px de marge au-dessus (comme le `row-gap` de la grille) et 60px de padding en dessous.

### Logo

Sur `info.html`, le logo est un lien vers `index.html` (`.logo a` : couleur héritée, sans soulignement). Il passe par la même transition de page que la nav. Sur `index.html`, le logo n'est pas cliquable.

### Page Info

- **Mise en page** : `.info-layout` est une grille à 12 colonnes (`column-gap: 40px`, même logique que Work). Le texte (`.hero`) occupe les colonnes 1 à 5, l'image (`figure.info-img`, `img/info.jpg`) les colonnes 8 à 12, alignée en haut du texte. Sous 768px, tout passe en pleine largeur et l'image se place sous le texte.
- **Stabilo** : le texte entre `<b>` est surligné en jaune pâle (`--highlight`). Rien n'est surligné au chargement ; la première animation démarre 1,5s après (fondu d'entrée de la page 0,8s + pause), puis un `<b>` s'anime toutes les 3,5s, à tour de rôle. Le fond se trace de gauche à droite (1,4s), reste visible 1,2s, puis s'efface dans le même sens (0,9s). Les durées de tracé/effacement sont dans `style.css` (`b.is-on`, `b.is-erasing`), le rythme et le délai initial dans le script en bas de `info.html`.

### Embed YouTube (facade cliquable)

Pour éviter d'afficher l'UI YouTube avant le clic : chaque vidéo est un `<button class="yt-facade" data-yt-id="...">` contenant juste une `<img>` (miniature `https://img.youtube.com/vi/ID/sddefault.jpg`). Au clic, un script remplace le bouton par un vrai `<iframe>` YouTube avec `autoplay=1`.

- Par défaut, utiliser `sddefault.jpg` (et non `maxresdefault.jpg`, qui renvoie 404 pour certaines vidéos).
- Pour certaines vidéos, la vignette est une image locale dans `img/` (`<img src="img/Xxx.jpg">`) à la place de la miniature YouTube (9 projets : Major Lazer, Pharrell, Jamie xx, Drake, Lana Del Rey, Woodkid - I love You, Is Tropical, Tyga, Dizzee Rascal).
- Le ratio du conteneur `.thumb` (`style="--ratio:..."`) doit correspondre au ratio natif de la vidéo, sinon YouTube ajoute des bandes noires. Valeurs utilisées : `1.78` (16:9, la majorité) et `2.45` (2 vidéos en cinémascope).

Les 18 projets de la page Work sont tous de vraies vidéos, dans cet ordre : Ibeyi - Aset, Ibeyi - Moshpit, Ibeyi - Offerings, Kenzo - Pre Fall 2016, The Blaze - Territory, Major Lazer - Get Free, Pharrell Williams - Happy, Jamie xx - Gosh, Louis Vuitton - Journey home for the holidays, Drake - Energy, Isabel Marant - SS23, Skrillex - Doompy Poomp, Lana Del Rey - Born to Die, Woodkid - I love You, Woodkid - The Golden age, Is Tropical - Dancing Anymore, Tyga - Bugatti, Dizzee Rascal - Couple of Stacks. Les IDs YouTube sont dans les `data-yt-id` de `index.html`.

## Décisions de design notables

- Aucun footer (supprimé volontairement des deux pages) ; le seul élément de fin de page sur Work est le "31" centré.
- Les rectangles gris (`--placeholder`, `.thumb` vide) restent le comportement voulu pour tout projet sans média, mais il n'y en a plus actuellement.
- Le texte de la page Info est le vrai texte (plus de lorem ipsum).

## Reste à faire / idées

- Supprimer `info old.html` s'il ne sert plus.
- Ajouter un `.gitignore` (au moins `.DS_Store`).
- `img/info.jpg` ne fait que 640px de large : le remplacer par une version plus grande pour un rendu net sur écran Retina.
- Le lien "Contact" de la nav d'origine a été remplacé par "Insta" (pas de page contact pour l'instant).
- Vérifier le rendu sur Safari iOS réel (comportement mobile de la grille, transitions, mise en page de la page Info).
