# Les Media Queries

Les media queries en CSS sont des outils puissants permettant d'adapter le style d'une page web aux caractéristiques du dispositif d'affichage, comme la largeur de l'écran. La syntaxe `@media` permet de créer des règles conditionnelles dans les feuilles de style.

## Syntaxe de Base

La syntaxe de base d'une media query utilisant `min-width` est comme suit :

```css
@media (min-width: 600px) {
  /* Règles CSS */
}
```

Cela signifie que les règles CSS à l'intérieur du bloc ne seront appliquées que si la largeur de la fenêtre d'affichage est d'au moins 600 pixels.

## Exemples

### Utilisation avec `columns`

Les propriétés CSS `column-count` et `column-gap` permettent de diviser le contenu d'un élément en plusieurs colonnes. Voici comment vous pouvez les ajuster en fonction de la largeur de l'écran :

```css
/* Style par défaut */
.content {
  column-count: 1;
  column-gap: 20px;
}

/* Sur écrans larges */
@media (min-width: 768px) {
  .content {
    column-count: 2;
  }
}

/* Sur écrans encore plus larges */
@media (min-width: 1024px) {
  .content {
    column-count: 3;
  }
}
```

Dans cet exemple, le contenu s'affichera en une seule colonne par défaut, puis passera à 2 colonnes si la largeur de l'écran atteint 768 pixels, et à 3 colonnes à partir de 1024 pixels de largeur.

### Ajustement de la taille d'un `h1`

Vous pouvez également utiliser les media queries pour ajuster la taille de la police d'un élément `h1` selon la largeur de l'écran :

```css
/* Style par défaut */
h1 {
  font-size: 24px;
}

/* Sur écrans moyens */
@media (min-width: 600px) {
  h1 {
    font-size: 28px;
  }
}

/* Sur écrans larges */
@media (min-width: 900px) {
  h1 {
    font-size: 32px;
  }
}
```

Ici, la taille de la police du `h1` augmente à mesure que la largeur de l'écran s'agrandit, améliorant ainsi la lisibilité sur les grands écrans.

Les media queries sont essentielles pour le design responsive, permettant aux développeurs de créer des expériences utilisateur adaptées à une large gamme d'appareils.
