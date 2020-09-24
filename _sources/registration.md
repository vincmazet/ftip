# Registration

% =============================================================================================== %

## Introduction

Le recalage de deux images consiste à trouver la correspondance entre deux images représentant la même zone.
Les figures ci-dessous présentent quelques applications dans lesquelles le recalage est indispensable.

```{figure} figs/registration/vincent4.png
---
height: 250px
name: F:registration:remote-sensing
---
Recalage d'images de télédétection :
comment combiner ces deux acquisitions différentes de la même zone pour former une grande image ?
```

```{figure} figs/registration/vincent6.png
---
height: 250px
name: F:registration:medical
---
Recalage en imagerie médicale :
pour détecter des changements entre deux examens,
il faut que les images soient recalées (le patient n'est jamais exactement dans la même position).
```

```{figure} figs/registration/vincent13.png
---
height: 300px
name: F:registration:mosaicing
---
Mosaïque d'image (_image mosaicing_) :
combiner plusieurs images pour en faire une seule grande, en photographie.
```

La {numref}`F:registration:bigpicture` représente le schéma classique pour effectuer le recalage entre deux images.
Tout l'enjeu est d'estimer la transformation spatiale qui fait correspondre deux images.
Les différentes étapes représentées dans cette figure sont détaillées dans la suite de ce chapitre.

```{figure} figs/registration/bigpicture.png
---
height: 300px
name: F:registration:bigpicture
---
Schéma classique du recalage de deux images.
```

% Several scenarios
% •   Monomodal   vs  mul6modal   registra6on
% •   Registra6on of  2D/2D,  2D/3D,  3D/3D,  3D+t,   nD/nD   images
% •   Global  vs  local   /   rigid   vs  deformable  transforma6on

% =============================================================================================== %

## Caractéristiques de recalage

La déformation à appliquer sur l'image source est basée sur des caractéristiques de cette image.
Le choix des caractéristiques utilisées conduit à l'une des deux méthodes de recalage décrite ci-après.

### Approche iconique

L'approche iconique (_intensity-based registration_)
se base directement sur l'intensité des pixels de l'image.
Ainsi, toute l'information contenue dans l'intensité des pixels est utilisée.
À la place de l'image elle-même, on peut aussi utiliser une de ses transformations (Fourier, ondelette, gradient...).

L'avantage de l'approche iconique est quelle est relativement bien automatisée.

Les inconvénients de l'approche iconique sont
le coût calculatoire important et la mémoire nécessaire,
mais elle est également sensible au bruit et aux variations d'intensité entre les images.

### Approche géométrique

Dans l'approche géométrique (_feature-based registration_),
la déformation à appliquer à l'image source n'est pas définie à partir des intensités des pixels,
mais à partir de primitives particulières de l'image.
Ces primitives peuvent être :
* intrinsèques à l'image : on se base sur les coins, les contours des objets, etc.
* extrinsèques à l'image : on se base sur des marqueurs insérés sur les objets.
Il faut ensuite apparier les primitives (_feature matching_).

```{figure} figs/registration/vincent18.png
---
height: 200px
name: F:registration:feature-based
---
Appariement de primitives entre deux images.
```

Les avantages de l'approche géométrique sont d'utiliser peu de données de l'image,
impliquant un coût calculatoire faible.

Les inconvénients de l'approche géométrique sont que :
* l'appariement des primitives peut être difficile à effectuer,
* la qualité du recalage dépend de la précision de l'extraction des primitives,
* la précision du recalage n'est garantie qu'au voisinage des primitives.

% =============================================================================================== %

## Modèle de déformation

Concrètement, l'image source est déformée grâce à une transformation mathématique qui peut être linéaire ou non.

Une transformation linéaire s'écrit :

$$
  p' = M p
$$

où
* $p = [x \; y \; 1]^T$ regroupe les coordonnées du pixel $(x,y)$ de l'image source,
* $p' = [x' \; y' \; 1]^T$ regroupe les coordonnées du pixel $(x',y')$ de l'image déformée,
* $M \in \mathbb{R}^{3\times3}$ est la matrice de transformation.

Par exemple, si la matrice de déformation est

$$
  M = \begin{bmatrix}
    1 & 0 & t_x \\
    0 & 1 & t_y \\
    0 & 0 & 1
  \end{bmatrix}
$$

alors

$$
  p' = \begin{bmatrix}
    x' \\
    y' \\
    1
  \end{bmatrix}
  = \begin{bmatrix}
    x + t_x \\
    y + t_y \\
    1
  \end{bmatrix}
$$

Les coordonnées du pixel $p'$ de l'image transformée
correspondent à celle du pixel $p$ de l'image source
après une translation de de $t_x$ pixels en $x$ et $t_y$ pixels en $y$.

Les exemples ci-dessous représentent l'effet de déformations particulières
sur l'image représentée {numref}`F:registration:lena-defnull`.

```{figure} figs/registration/lena-defnull.png
---
scale: 50%
name: F:registration:lena-defnull
---
Image Lena.
```

### Déformation rigide

Une déformation rigide est une transformation linéaire
définie avec 3 paramètres : $\theta=\{\alpha,t_x,t_y\}$.
La matrice de déformation s'écrit :

$$
  M\!=\!\begin{pmatrix}
    \cos\alpha & \sin\alpha & t_x \\
    -\sin\alpha & \cos\alpha & t_y \\
    0 & 0 & 1
  \end{pmatrix}
$$

```{figure} figs/registration/lena-defrigide.png
---
scale: 50%
name: F:registration:lena-defrigide
---
Déformation rigide.
```

### Déformation affine

Une déformation affine est une transformation bilinéaire définie
avec 6 paramètres : $\theta=\{m_{11}, m_{12}, m_{13}, m_{21}, m_{22}, m_{23}\}$.
La matrice de déformation s'écrit :

$$
  M\!=\!\begin{pmatrix}
    m_{11} & m_{12} & m_{13} \\
    m_{21} & m_{22} & m_{23} \\
    0 & 0 & 1
  \end{pmatrix}
$$


```{figure} figs/registration/lena-defaffine.png
---
scale: 50%
name: F:registration:lena-defaffine
---
Déformation affine.
```

### Déformation perspective

Une déformation perspective est une transformation linéaire définie
avec 9 paramètres : $\theta=\{m_{11}, m_{12}, m_{13}, m_{21}, m_{22}, m_{23}, m_{31}, m_{32}, m_{33}\}$.
La matrice de déformation s'écrit :

$$
  M\!=\!\begin{pmatrix}
    m_{11} & m_{12} & m_{13} \\
    m_{21} & m_{22} & m_{23} \\
    m_{31} & m_{32} & m_{33}
  \end{pmatrix}
$$

```{figure} figs/registration/lena-persp.png
---
scale: 50%
name: F:registration:lena-persp
---
Déformation perspective.
```

### Déformation non linéaire

On peut définir tout autre type de déformation sans passer par une transformation linéaire.
Ainsi, l'utilisation de fonctions 2D spécifiques (polynôme, sinusoïde, spline, ondelette...)
ou carrément d'un champ de déformation non paramétrique est envisageable.

Il peut alors être nécessaire d'introduire des contraintes sur le modèle de déformation
(préservation de la topologie, douceur, symétrie...).

```{figure} figs/registration/lena-deffnonlin.png
---
scale: 50%
name: F:registration:lena-deffnonlin
---
Déformation non linéaire.
```

% =============================================================================================== %

## Interpolation de l'image recalée

L'{ref}`C:interpolation` a été vue dans le chapitre précédent et ne sera pas détaillé ici.

% L'interpolation consiste à déterminer les valeurs de l'image transformée à partir de ceux de l'image initiale.
%
% interp1
%
% Problème : les pixels de l'image transformée ne correspondent pas toujours à un pixel de l'image de l'image cible.
%
% interp2
%
% L'astuce est de considérer la transformation inverse et de déterminer la valeur des pixels de l'image transformée
% en fonction de ceux de l'image initiale.
%
% interp3
%
% Méthodes d'interpolation :
% * plus proche voisin (_nearest neighbor_)
% * bilinéaire,
% * B-spline,
% * sinc
% * ...

% =============================================================================================== %

## Critère de similarité

Le critère de similarité $E(\theta)$ représente la distance (au sens mathématique)
entre l'image de référence $g$ et l'image déformée $f$
(obtenue an appliquant le modèle de déformation de paramètres $\theta$ sur l'image source $f$).
Cette distance est minimale lorsque les deux images se superposent au mieux,
c'est-à-dire lorsque la similarité entre l'image de référence $g$ et l'image déformée $f\,'$ est maximale.

L'objectif du recalage est de trouver les paramètres $\theta$ qui minimisent le critère de similarité $E(\theta)$ :
c'est donc un problème d'optimisation qui sera abordé dans la {ref}`section suivante <C:registration:optimisation>`.

Le choix de $E$ dépend du choix de l'approche choisie.

### Approche iconique

Dans le cas d'une approche iconique (basée sur l'intensité des pixels),
on utilisera un critères de similarité dit _dense_.
Il existe plusieurs hypothèses sur les liens entre les intensités des deux images.
Dans le cas le plus simple, on suppose que les intensités des pixels
sont égales à un bruit additif gaussien près, donc :

$$
  E(\theta) = \sum_{m=1}^M \sum_{n=1}^N \big(f\,'(m,n) - g(m,n)\big)^2
$$

### Aapproche géométrique

Dans le cas d'une approche géométrique (basée sur des primitives),
on utilisera une distance entre ces primitives.

* Par exemple, dans le cas où les primitives sont des pixels particuliers,
  on peut considérer la distance entre ces pixels avec la norme euclidienne :

  $$
    E(\theta) = \sum_{n=1}^N (x_n-x'_n)^2 + (y_n-y'_n)^2
  $$

  où $(x_n,y_n)$ et $(x'_n,y'_n)$ sont les coordonnées des pixels appariés.

* Un autre exemple est le cas où les primitives sont des courbes.
  L'algorithme ICP (_Iterative Closest Point_) peut être utilisé pour déterminer une distance entre ces deux courbes, définie comme

  $$
    E(\theta) = \sum_{n=1}^N d_n^2
  $$

  où $d_n$ est la distance entre chaque point de la courbe 1 avec le point le plus proche de la courbe 2.

  ```{figure} figs/registration/icp.png
  ---
  height: 150px
  name: F:registration:icp
  ---
  Distance entre deux courbes avec l'algorithme ICP.
  ```

  % TODO : ref

% =============================================================================================== %

(C:registration:optimisation)=
## Optimisation du critère de similarité

% TODO : introduire des réfs

Comme on l'a dit, précédemment, on cherche les valeurs des paramètres $\theta$
de la transformation qui minimise $E(\theta)$.
Mathématiquement, le problème s'écrit :

$$
  \hat{\theta} = \arg \min_{\theta} E(\theta)
$$


```{figure} figs/registration/optim.png
---
height: 200px
name: F:registration:optim
---
Principe de l'optimisation d'un critère $E$ :
exemple pour $\theta=\alpha$ dans le cas d'une simple rotation.
```

Il existe énormément de méthodes d'optimisation dont la description dépasse le cadre de ce cours :
* Solution explicite (en annulant la dérivée de $E$),
* Recherche exhaustive (toutes les possibilités sont testées),
* Méthodes déterministes :
  algorithme du simplexe, descente de gradient, gradient conjugué,
  algorithme de Levenberg-Marquardt, etc.
* Méthodes stochastiques :
  recuit simulé, algorithmes génétiques, gradient stochastique, etc.

% solution explicite : méthode procustéenne, (_exact point matching_)

### Approches hiérarchiques

L'idée des approches hiérarchiques (_hierarchical / coarse to fine approaches_)
est de décomposer le problème initial en plusieurs petits problèmes de complexité moindre.
Cela a tendance à réduire le risque de convergence vers un minimum local et à accélérer le calcul.

* Dans le cas de données complexes, on adoptera une approche par multi-résolution :
  on commence par effectuer l'optimisation sur une image très sous-échantillonnée (niveau 0),
  et refaire l'optimisation à différentes échelles jusqu'à l'image originale.
  Chaque algorithme d'optimisation est initialisé avec la valeurs de $\theta$ obtenue
  au niveau précédent.

  ```{figure} figs/registration/multiresolution.png
  ---
  height: 200px
  name: F:registration:multiresolution
  ---
  Exemple : pyramide gaussienne.
  ```

* Dans le cas d'un Modèle de déformation complexe,
  on privilégiera une complexification de ce modèle.
  Par exemple, on pourra commencer par considérer une déformation rigide,
  qui sera affinée ensuite en déformation affine voire non rigide.
  On peut aussi utiliser une approche multi-échelle (_multiscale approach_)
  illustrée {numref}`F:registration:multiscale`.

    ```{figure} figs/registration/multiscale.png
    ---
    height: 100px
    name: F:registration:multiscale
    ---
    Approche multi-échelle.
    ```

% =============================================================================================== %
