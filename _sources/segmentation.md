(C:segmentation)=
# Segmentation

% =============================================================================================== %

## Introduction

La segmentation consiste à partitionner une image $f$ selon un _critère d'homogénéité_.
« Partitionner » signifie que l'image est divisée en plusieurs régions $R_i$
qui sont à la fois disjointes et telles que l'ensemble de ces régions recouvrent l'intégralité de l'image.
Les pixels d'une région vérifient le critère d'homogénéité,
mais les pixels de deux régions adjacentes ne le vérifient pas.

Les figures ci-après présentent plusieurs exemples de segmentation.

```{figure} figs/segmentation-examples.png
---
width: 500px
name: F:segmentation:examples
---
Exemples de segmentation.
```

Ce chapitre présente plusieurs méthodes de segmentation,
et termine sur la façon de mesurer la qualité de la segmentation.
Mais avant tout, il est utile de définir quelques termes liés à la relation qui peut exister entre les pixels.


**Connexité**

La connexité est la façon dont sont définis les voisins d'un pixel.
En général, on n'utilise que l'une des deux connexités suivantes :
* la 4-connexité : un pixel possède quatre voisins (en haut, en bas, à gauche, à droite),
* la 8-connexité : un pixel possède huit voisins (les quatre précédents et ceux sur les diagonales).

```{figure} figs/connexity.png
---
height: 200px
name: F:segmentation:connexity
---
4-connexité (à gauche) et 8-connexité (à droite). Les pixels en gris sont les voisins du pixel $(m,n)$.
```


**Composante connexe**

Une composante connexe (_connected component_) est un groupe de pixels
tel qu'on puisse aller d'un pixel de ce groupe à un autre pixel de ce groupe
en passant par des pixels du même groupe voisins entre eux.

Ainsi, dans la {numref}`F:segmentation:connected-component`,
le nombre de composantes connexes est égal à 5 si on considère un voisinage en 4-connexité,
ou à 4 si on considère un voisinage en 8-connexité.

```{figure} figs/connected-component.png
---
height: 150px
name: F:segmentation:connected-component
---
Le nombre de composantes connexes (entourées d'un trait de couleur) dépend de la connexité considérée.
```


**Remarques**

* Chaque région $R_i$ de la segmentation est une composante connexe.

* Le résultat de la segmentation n'est pas unique :
  il dépend du critère d'homogénéité choisi, de la méthode de segmentation utilisée, de l'initialisation, etc.

* Une segmentation peut s'interpréter comme un graphe,
  dans lequel les nœuds correspondent aux régions $R_i$ et les liens représentent l'adjacence entre régions voisines.
  
  ```{figure} figs/connected-component-graph.png
  ---
  height: 150px
  name: F:segmentation:connected-component-graph
  ---
  Exemple de segmentation et son graphe associé.
  ```


% =============================================================================================== %

## Histogram thresholding

### Binarisation

Une méthode très simple de segmentation consiste à associer à chaque pixel de l'image $f$
une valeur binaire qui dépend de l'intensité des pixels et d'un seuil $T$ :

$$
  g(m,n) =
  \begin{cases}
    1 & \text{si}\, f(m,n)\geqslant T, \\
    0 & \text{si}\, f(m,n)< T
  \end{cases}
$$

Cette méthode, appelée « binarisation », effectue une segmentation en deux classes
à partir de l'intensité des pixels d'une image à niveau de gris (cf. {numref}`F:segmentation:binarize`).

```{figure} figs/segmentation-binarize.png
---
height: 250px
name: F:segmentation:binarize
---
Exemple de binarisation d'une image de bactéries.
```

Le résultat de la segmentation dépend de $T$, comme on le voit {numref}`F:segmentation:threshold1`.

```{figure} figs/segmentation-threshold1.png
---
height: 200px
name: F:segmentation:threshold1
---
Résultat de la segmentation en fonction du choix du seuil.
```

L'histogramme s'avère être un outil intéressant pour le choix du seuil.
Dans l'exemple illustré {numref}`F:segmentation:threshold2`, le choix du seuil est facile à partir de l'histogramme :
il suffit de choisir $T$ entre les deux modes de ce dernier.

```{figure} figs/segmentation-threshold2.png
---
height: 200px
name: F:segmentation:threshold2
---
Segmentation binaire avec un seuil choisi par rapport à l'histogramme.
```

Mais dans d'autres cas, le choix du seuil est moins évident,
comme c'est le cas {numref}`F:segmentation:threshold3`
(il faut alors choisir un seuil comme dans la {numref}`F:segmentation:threshold4`).

```{figure} figs/segmentation-threshold3.png
---
height: 200px
name: F:segmentation:threshold3
---
Segmentation binaire avec un seuil choisi par rapport à l'histogramme.
```

```{figure} figs/segmentation-threshold4.png
---
height: 200px
name: F:segmentation:threshold4
---
Segmentation binaire avec un seuil choisi par rapport à l'histogramme.
```


### Méthode de Otsu

La méthode de Otsu (1979) permet de déterminer le seuil $T$ qui minimise la « variance intra-classe » $\sigma_w^2(T)$.
Cette variance intra-classe est la moyenne pondérée des variances $\sigma_1$ et $\sigma_2$ de chaque classe
(les classes étant les parties de l'histogramme délimitées par $T$, comme illustré {numref}`F:segmentation:celine20`):

$$
  T = \arg \min_T \sigma_w^2(T) = \arg \min_T q_1(T)\sigma_1^2(T) + q_2(T)\sigma_2^2(T)
$$

```{figure} figs/celine20.png
---
height: 300px
name: F:segmentation:celine20
---
Notations utilisées.
```

En considérant que les intensités $i$ sont à valeurs dans $\{0,...,L-1\}$
et que $h$ est l'histogramme de l'image, on a formellement :

* Pour la classe 1 :
  - Proportion :    $\displaystyle q_1(T) = \frac{1}{MN} \sum_{i = 0}^{T} h(i)$
  - Moyenne :       $\displaystyle m_1(T) = \frac{1}{MN}\frac{1}{q_1(T)} \sum_{i = 0}^{T} i h(i)$
  - Variance :      $\displaystyle \sigma^2_1(T) = \frac{1}{MN}\frac{1}{q_1(T)} \sum_{i = 0}^{T}(i\!-\!m_1(T))^2 h(i)$

* Pour la classe 2 :
  - Proportion :    $\displaystyle q_2(T) = \frac{1}{MN}\sum_{i = T+1}^{L-1} h(i)$
  - Moyenne :       $\displaystyle m_2(T) = \frac{1}{MN}\frac{1}{q_2(T)} \sum_{i = T+1}^{L-1} i g(i)$
  - Variance :      $\displaystyle \sigma^2_2(T) = \frac{1}{MN}\frac{1}{q_2(T)} \sum_{i = T+1}^{L-1}(i\!-\!m_2(T))^2 h(i)$


L'algorithme est simple : il suffit de calculer la variance intra-classe $\sigma_w^2(T)$ pour tous les seuils $T = \{0,...,L-1\}$,
et de retenir finalement le seuil $T$ qui minimise $\sigma_w^2(T)$.

Par ailleurs, remarquez que la variance $\sigma^2$ des intensités de l'image s'écrit :

$$
  \sigma^2 = \sigma_w^2 + \sigma_{1,2}^2
$$

où $\sigma_{1,2}^2$ est la « variance inter-classe » (variance pondérée des moyennes de chaque classe).
Ainsi, minimiser la variance intra-classe $\sigma_w^2$ est équivalent à maximiser la variance inter classe $\sigma_{1,2}^2$
(puisque $\sigma^2$ reste constant).
Cela signifie que construire deux groupes de pixels qui se ressemblent
revient à construire deux groupes de pixels très dissemblables.


### Seuillage multiple

Lorsque plusieurs modes sont visibles sur l'histogramme,
il est possible d'utiliser plusieurs seuils pour aboutir à plusieurs classes :

```{figure} figs/segmentation-multiple-thresholds.png
---
height: 200px
name: F:segmentation:multiple-thresholds
---
Seuillage multiple.
```

En particulier, la méthode de Otsu peut être étendue à plusieurs seuils,
mais la complexité calculatoire augmente grandement avec le nombre de classes !

<!-- La variation d'illumination ne permet pas de seuiller correctement l'image. Plusieurs solutions sont possibles.
TODO : afficher :
- image originale + son histogramme + son seuillage
- fond
- image sans fond + son histogramme + son seuillage
[image : seuillage_variation_illumination, \imgref{Gonzalez $\&$ Woods}

Éclairage $g$ connu : on utilise un modèle paramétrique pour le décrire et on corrige l'image avant le seuillage :
$$
\forall (m,n), \quad  h(m,n) = \frac{f(m,n)}{g(m,n)}
$$
* Éclairage $g$ inconnu : seuillage local par exemple
  \includegraphics[width=8.5cm]{seuillage_bloc}
  \imgref{Gonzalez $\&$ Woods}
\pnode(3.8,1.2){A}
\rput[lt](2,0.3){\rnode{B}Attention : zone à une seule classe ($\Rightarrow$ test de la variance)}
\ncline[linecolor=gray]{<-}{A}{B}

Influence du bruit
Ajout d'un bruit gaussien sur l'image $\Rightarrow$ convolution de l'histogramme de l'image par une gaussienne.
  \includegraphics[width=10cm]{seuillage_bruit}
  \imgref{Gonzalez $\&$ Woods}
Solutions possibles :
* Débruiter l'image initiale (filtre gaussien, filtre moyenneur, méthode de débruitage, ...)
* Filtrer l'image seuillée (opérateurs morphologiques, filtre médian, ...)
  \only<2>{\rput[c](.5\linewidth,-2){\includegraphics[width=8cm]{vincent72}}}
* Incorporer de l'information spatiale dans la méthode de segmentation.
  \includegraphics[width=10cm]{seuillage_filtre_median} -->

% =============================================================================================== %

## Classification

Le seuillage s'applique sur une image monochrome, pour laquelle il est facile de définir un seuil à partir des modes de l'histogramme.
Mais pour segmenter une image multibande (par exemple une image couleur),
il n'est pas possible de s'appuyer sur l'histogramme de l'image car il est possède maintenant plusieurs dimensions.
Un pixel d'une image à $B$ bandes est ainsi représenté par un vecteur à valeurs dans $\mathbb{R}^B$.

```{figure} figs/segmentation-classification.png
---
height: 200px
name: F:segmentation:clasification
---
Représentation dans $\mathbb{R}^B$ d'un pixel d'une image.
```

Le principe des méthodes de classification (ou plus exactement de coalescence, en anglais : _clustering_)
est de regrouper les pixels en groupes homogènes.


### Algorithme des k-moyennes

L'algorithme des k-moyennes (_k-means_) (Steinhaus 1957, MacQueen 1967)
est une méthode itérative qui affecte chaque point de l'espace $\mathbb{R}^B$ (chaque pixel, donc)
à un groupe particulier (_clusters_).
Le nombre $K$ de groupes est choisi par l'utilisateur.

L'algorithme est le suivant :
* Initialisation aléatoire de $K$ centroïdes
* Répéter tant que les centroïdes varient :
  - Pour chaque point :
    - Calcul des distances du point à tous les centroïdes
     - Affectation du point au groupe le plus proche
  - Calcul du centroïde de chacun des groupes

La {numref}`F:segmentation:kmeans-algo` illustre cet algorithme,
dans le cas simple d'une image à deux bandes (espace à deux dimensions)
à segmenter en $K=2$ classes (deux couleurs, ici rouge et vert).

```{figure} figs/segmentation-kmeans-algo.gif
---
height: 200px
name: F:segmentation:kmeans-algo
---
Illustration de l'algorithme des k-moyennes.
```

La {numref}`F:segmentation:kmeans-result` donne le résultat de l'algorithme des k-means sur une image.

```{figure} figs/segmentation-kmeans-result.png
---
height: 200px
name: F:segmentation:kmeans-result
---
Exemple d'application de l'algorithme des k-means sur l'image de gauche (au centre : $K=2$ classes, à droite : $K=4$ classes).
```

Les avantages de l'algorithme des k-means sont :
* méthode simple
* implémentation facile
* méthode généralement rapide
* classes de variance conditionnelle minimale
* fonctionne correctement lorsque les clusters sont sphériques
  ```{image} figs/vinc51-1.png
  :height: 70px
  ```
  
Les inconvénients de l'algorithme des k-means sont :
* nécessite de connaître le nombre de classes
* sensible aux minima locaux, donc à l'initialisation
* peut être lent en grande dimension
* échoue pour des structures non sphériques
  ```{image} figs/vinc51-2.png
  :height: 70px
  ```
* sensible aux valeurs aberrantes
  ```{image} figs/vinc51-3.png
  :height: 70px
  ```

<!-- ### Modèles paramétriques


L'histogramme de l'image est modélisé par un mélange de lois \eng{mixture model} :
on dispose d'un modèle paramétrique représentatif des classes présentes dans l'image.

  \includegraphics[width=10cm]{vincent52}

* Lois : souvent gaussiennes \eng{GMM : Gaussian mixture model}.
* Extension possible à plusieurs dimensions

Deux étapes :
\begin{enumerate}
* Estimation des paramètres des lois
  {\color{gray}Poids $\Pi_k$, moyennes $\mu_k$, écart-types $\sigma_k$}
    \includegraphics[width=6cm]{vincent54}
* Classification
  {\color{gray}Associer à chaque intensité la classe la plus représentative}
\end{enumerate}

\paragraph{{\color{unistra}$\blacksquare$\hspace*{-.6em}\scriptsize\sf\raisebox{.5mm}{\color{white}1}}\; Estimation}

$$
  \forall i,\qquad
  p(h(i)|\theta) = \sum_{k=1}^K \frac{\Pi_k}{\sqrt{2\pi\sigma_k^2}} \exp\left(-\frac{(h(i)-\mu_k)^2}{2\sigma_k^2}\right)
$$

où $K$ est le nombre de classes
et $\theta$ regroupe les paramètres inconnus des lois :
$\theta=[\Pi_1,...,\Pi_K,\mu_1,...,\mu_K,\sigma_1,...,\sigma_K]$.

Estimation des paramètres au sens du maximum de vraisemblance :

$$
  \hat{\theta}^\text{MV} = \arg \max_{\theta} \prod_i p(h(i)|\theta)
$$

Méthode de résolution : algorithme EM, algorithmes MCMC, ...

\paragraph{{\color{unistra}$\blacksquare$\hspace*{-.6em}\scriptsize\sf\raisebox{.5mm}{\color{white}2}}\; Classification}

  \includegraphics[width=6cm]{vincent54}

Chaque pixel est affecté à la classe dont il maximise la loi :

$$
  f_\text{seg}(m,n) = \arg \max_{k\in\{1,...,K\}}
    \frac{\Pi_k}{\sqrt{2\pi\sigma_k^2}} \exp\left(-\frac{(f(m,n)-\mu_k)^2}{2\sigma_k^2}\right)
$$

\parbox{.45\textwidth}{\paragraph{k-moyennes}}
\parbox{.45\textwidth}{\paragraph{Mélange de gaussiennes}}

\parbox{.45\textwidth}{Estimation uniquement des $\mu_k$}
\parbox{.45\textwidth}{Estimation des $\mu_k$ et $\sigma_k$}

\parbox{.45\textwidth}{Sensible à l'initialisation}
\parbox{.45\textwidth}{Sensible à l'initialisation}

\parbox{.45\textwidth}{Sensible aux minima locaux}
\parbox{.45\textwidth}{Sensible aux minima locaux}

\parbox{.45\textwidth}{Nécessite de connaître le nombre de classes}
\parbox{.45\textwidth}{Nécessite de connaître le nombre de classes}

\parbox{.45\textwidth}{\centering\includegraphics[width=4cm]{vincent58-kmeans}}
\parbox{.45\textwidth}{\centering\includegraphics[width=4cm]{vincent58-gmm}} -->

% =============================================================================================== %

## Region-based methods

La limite fondamentale des méthodes précédentes est
de ne pas prendre en compte l'information de voisinage :
seule l'information de distribution des intensités est utilisée.

À l'inverse, les méthodes basées région sont capables d'agréger des pixels spatialement proches _et_ ayant des intensités similaires.
Nous allons voir deux méthodes basées régions :
* la croissance de région
* la méthode de décomposition/fusion


### Croissance de région

Le principe de la croissance de région (_region growing_) est, à partir d'un pixel initial (appelé « germe »),
d'étendre la région en y ajoutant les pixels du voisinage qui satisfont le critère d'homogénéité,
comme l'illustre la {numref}`F:segmentation:regiongrowing`.

```{figure} figs/segmentation-regiongrowing.png
---
height: 250px
name: F:segmentation:regiongrowing
---
Illustration de la croissance de région (le germe est indiqué par la croix rouge).
```

Le choix du germe peut se faire manuellement ou automatiquement
(par exemple en choisissant au hasard un pixel en dehors des zones de fort contraste).

Le critère de similarité est le suivant :
si un pixel $f(m,n)$ et une région $R$ sont suffisamment similaires, alors ils sont fusionnés ; sinon une nouvelle région est créée.
On peut utiliser par exemple le critère 

$$
  \left| f(m,n) - \mu_R \right| < T \sigma_R
$$

Ainsi, si le paramètre $T$ est élevé, il sera facile d'agréger des nouveaux pixels à la région.
Au contraire, si $T$ faible alors il sera plus difficile d'agréger des nouveaux pixels à la région.

<!-- Choix de la connexité : 4-voisinage ou 8-voisinage. -->

<!-- * $R$ : région segmentée, initialisée au germe
* $S$ : pixels à tester, initialisé au voisinage du germe (file FIFO : \emph{first in, first out})
Algorithme :
  \setlength{\fboxsep}{3mm}
  \colorbox{algobg}{\parbox{.9\textwidth}{
  tant que $S$ n'est pas vide :
  \albar $p$ est le premier pixel de la liste $S$
  \albar $p$ est retiré de $S$
  \albar si $p$ est homogène avec $R$ :
  \albar\albar ajout à $R$ de $p$
  \albar\albar ajout à $S$ des pixels du voisinage de $p$ qui ne sont pas dans
  \albar\albar\qquad $R$ et qui ne sont pas incompatibles.
  \albar sinon :
  \albar\albar $p$ est marqué comme incompatible.
 -->

La croissance de région ne fournit pas directement une partition de l'image,
mais permet de segmenter une ou plusieurs structures d'intérêt via la sélection de germes adaptés.
Pour segmenter une image en $K$ classes, il faudra donc $K$ germes.

<!-- Au moins deux points germes sont nécessaires :
  \imgbox{45mm}{eclairs}{Image originale}\qquad
  \imgbox{45mm}{eclair1}{Image segmentée} -->

<!-- Quelle segmentation est obtenue avec la plus grande valeur de $T$ ?
    \imgbox{40mm}{eclair1}{\only<1>{A}\only<2>{$T$ petit}\phantom{gT}}\qquad
    \imgbox{40mm}{eclair3}{\only<1>{B}\only<2>{$T$ grand}\phantom{gT}}% -->

<!-- % TODO : ??? \textcolor{red!70}{Rq : en cas d'utilisation de statistique globale pour le test d'homogénéité, l'ordre de traitement des pixels peut influencer le résultat final.} -->


### Décomposition/fusion

La méthode de décomposition/fusion (_split and merge_) fonctionne en deux étapes :
1. d'abord, l'image est décomposée successivement en régions
   si elles ne satisfont pas le critère d'homogénéité.
   Cela permet d'aboutir à une première partition de l'image ;
1. ensuite, les régions obtenues sont fusionnées si elles sont adjacentes et qu'elles vérifient le critère d'homogénéité.

<!-- \onslide<4->{Les représentations en arbre et par graphe permettent une représentation haut niveau de l'image.} -->


**Décomposition**

La décomposition est une procédure itérative.
Au départ, il n'y a qu'une seule région qui correspond à l'image toute entière.
À chaque itération, les régions qui ne vérifient pas le critère d'homogénéité sont divisées en quatre nouvelles régions de taille identique.
La procédure s'arrête lorsque les régions sont toutes homogènes ;
au pire, les régions les plus petites sont ainsi des pixels uniques.

On peut utiliser une représentation en quad-arbre (_quad-tree_) de cette décomposition :
c'est une arborescence dont chaque nœud représente une région et possède quatre fils,
la racine représente l'image entière.

```{figure} figs/segmentation-split-merge-quadtree.gif
---
height: 300px
name: F:segmentation:split-merge-quadtree
---
Illustration de la décomposition avec représentation en quad-arbre.
```

Finalement, la méthode de décomposition par quad-arbre fait apparaître des régions carrées sur l'image segmentée.
Le problème majeur de cette décomposition provient de la rigidité des divisions réalisées sur l'image,
mais au moins cela fournit une partition initiale de l'image.

**Fusion**

La partition de l'image obtenue avec la la représentation en quad-arbre
peut être vue comme un graphe d'adjacence (RAG : _region adjacency graph_).
C'est une nouvelle représentation, sous forme de graphe, dont :
* les nœuds correspondent à une région de l'image,
* les arêtes relient les nœuds correspondants à deux régions adjacentes (ayant une frontière commune).
La {numref}`F:segmentation:rag` donne un exemple de tel graphe.

```{figure} figs/vincent88.png
---
height: 200px
name: F:segmentation:rag
---
Une image segmentée et sa représentation sous forme de graphe d'adjacence.
```

Donc, à partir de ce graphe d'adjacence, les nœuds $R_1$ et $R_2$ voisins et dont le critère de similarité sur $R_1 \cup R_2$ est respecté
sont fusionnés (cf. {numref}`F:segmentation:split-merge-rag`).

```{figure} figs/segmentation-split-merge-rag.gif
---
height: 300px
name: F:segmentation:split-merge-rag
---
Illustration de la fusion avec représentation en graphe d'adjacence.
```

% =============================================================================================== %

## Watershed

La ligne de partage des eaux (_watershed_) considère l'image comme un carte topographique où :
* les régions de la segmentation sont les vallées
* les frontières entre régions sont les crêtes

Ainsi la {numref}`F:segmentation:watershed-moon` montre, pour une image de la Lune, ce que devrait être la carte topographique correspondante.
Cette carte est en fait la vue 3D de la norme du gradient de l'image.

```{figure} figs/segmentation-watershed-moon.png
---
height: 200px
name: F:segmentation:watershed-moon
---
Une image et son gradient (vu comme une image et comme un signal 3D, faisant apparaître le relief).
```

Le principe de la ligne de partage des eaux est donc :
1. de construire la carte d'élévation,
1. de remplir progressivement d'eau chaque bassin versant : l'eau apparaît tout en bas du relief,
1. de faire monter le niveau de l'eau,
1. lorsque deux bassins se rejoignent, la ligne de partage des eaux est marquée comme frontière.

La {numref}`F:segmentation:watershed-algo` schématise cet algorithme sur une coupe de l'image.

```{figure} figs/segmentation-watershed-algo.gif
---
height: 200px
name: F:segmentation:watershed-algo
---
Schématisation de l'algorithme de ligne de partage des eaux.
```

<!-- Algorithme :
 \setlength{\fboxsep}{3mm}
 \colorbox{algobg}{\parbox{.9\textwidth}{
  Calculer le gradient (ou le Laplacien) de l'image.
  Les pixels ayant l'intensité la plus faible forment les bassins \phantom{\albar}\quad versants initiaux.
  Pour chaque niveau d'intensité $i$ :
  \albar Pour chaque groupe de pixels d'intensité $i$ :\\
  \albar\albar Si adjacent à exactement une région existante :\\\albar\albar\quad ajouter ces pixels dans cette région.\\
  \albar\albar Si adjacent à plusieurs régions simultanément :\\\albar\albar\quad marquer comme ligne de partage des eaux.
  \albar\albar Sinon, commencer une nouvelle région. -->

Une des limites de cette méthode apparaît lorsqu'il y a beaucoup de minima locaux dans le gradient.
Dit autrement, il y a trop de bassins versants très petits, qui sont alors autant de régions dans la segmentation.
Pour limiter ce nombre, on peut :
* lisser (avec un filtre passe-bas) le gradient avant d'appliquer l'algorithme,
* choisir manuellement les bassins versants d'intérêt avec des marqueurs,
* ou fusionner les minima locaux.

% =============================================================================================== %

<!-- ## Snakes

Contours actifs

Principe : à partir d'un contour initial proche de l'objet à segmenter,
le contour évolue de manière itérative et cherche à converger
vers les zones de fort gradient (= contour) sous certaines contraintes (forme, longueur, etc.).

Le contour est modélisé par un ensemble de points (x_i,y_i)
qui se déplacent légèrement à chaque itération pour déformer le contour.

  \includegraphics[width=7cm]{vincent117-1}

Le contour cherche à minimiser une énergie (ou fonction coût)
qui mesure la qualité de la segmentation :

  E_\text{totale} = E_\text{interne} + \lambda E_\text{externe}

* Énergie interne : encourage certaines configurations de forme
  (régularité, élasticité, a priori de forme, ...)
* Énergie externe : encourage le modèle à converger vers les contours des objets
  (zones de fort gradient)

% Différents types d'énergie interne :
%   \includegraphics[width=8cm]{vincent115} -->

% =============================================================================================== %

## How to evaluate the segmentation?

Ce chapitre a présenté les principales méthodes de segmentation, mais il en existe beaucoup d'autres !
Il n'existe pas une méthode de segmentation meilleure que tous les autres, dans tous les cas : le résultat dépend entre autre de l'image elle-même.
Par conséquent, il est intéressant d'évaluer, pour le type d'image que l'on traite, la qualité de la segmentation.
Pour cela, on peut utiliser différents critères, définis ci-après.
En plus de l'image à segmenter, on a également besoin du résultat attendu, qu'on appelle « vérité terrain » (_ground truth_).

Imaginons, dans le cas d'une segmentation binaire,
que la vérité terrain et la segmentation obtenue sont les images de la {numref}`F:segmentation:eval:gt-est`.
Chaque image possède donc deux zones : l'objet segmenté (représenté en blanc) et le fond (en noir).
Alors, on peut définir quatre types de zones (cf. {numref}`F:segmentation:eval-zones`) :
- les vrais positifs (VP) représentent les pixels considérés comme étant dans l'objet et étant réellement dans l'objet,
- à l'inverse, les vrais négatifs (VN) sont les pixels hors de l'objet à la fois dans la segmentation et la vérité terrain,
- les faux positif (FP) sont les pixels considérés par la segmentation dans l'objet, mais qui en vrai n'en font pas partie,
- enfin, les faux négatif (FN) sont les pixels de l'objet que la segmentation a classé en dehors.

```{figure} figs/segmentation-eval-gt-est.png
---
height: 200px
name: F:segmentation:eval:gt-est
---
Vérité terrain $f^*$ (à gauche) et segmentation obtenue $f$ (à droite).
```

```{figure} figs/eval-zones.png
---
height: 200px
name: F:segmentation:eval-zones
---
Définition des vrais positifs (VP), faux positifs (FP), vrais négatifs (VN) et faux négatifs (FN).
```

À partir de ces quatre quantités, on peut utiliser l'un ou l'autre des critères suivants :
* la sensibilité :

  $$
  \frac{\text{VP}}{\text{VP}+\text{FN}},
  $$
  
* la spécificité :

  $$
  \frac{\text{VN}}{\text{VN}+\text{FP}},
  $$
  
* le coefficient de Dice :

  $$
  \frac{2\,\text{VP}}{2\,\text{VP}+\text{FP}+\text{FN}} = \frac{2\,|f\, \cap f\,^*|}{|f\,| + |f\,^*|},
  $$
  
* le coefficient de Jaccard :

  $$
  \frac{\text{VP}}{\text{VP}+\text{FP}+\text{FN}} = \frac{|f\, \cap f\,^*|}{|f\, \cup f\,^*|}.
  $$

% =============================================================================================== %

## Conclusion

En conclusion, nous avons vu que la segmentation consiste à diviser l'image en plusieurs régions homogènes.
L'homogénéité d'une région est basée sur la couleur, la texture, les contours...
Les méthodes de segmentation sont très diverses, et nous n'en avons vu que quelques unes.
Parmi les autres méthodes existantes, citons les contours actifs (_snakes_),
les ensembles de niveaux (_level sets_), les modèles markoviens, etc.


<!--   \bibitem[Achanta  et coll. 2012]{Achanta12}
  R. Achanta, A. Shaji, K. Smith, A. Lucchi, P. Fua, S. Süsstrunk,
  \og{}SLIC Superpixels Compared to State-of-the-art Superpixel Methods \fg{},
  \emph{IEEE Transactions on Pattern Analysis and Machine Intelligence}, 34(11), p. 2274--2282, 2012.

  \bibitem[Fukunaga \& Hostetler 1975]{Fukunaga75}
  K. Fukunaga, L.D. Hostetler,
  \og{}The Estimation of the Gradient of a Density Function, with Applications in Pattern Recognition\fg{},
  \emph{IEEE Transactions on Information Theory}, 21(1) p. 32--40, 1975.

  \bibitem[MacQueen 1967]{MacQueen67}
  J.B. MacQueen,
  \og{}Some Methods for classification and Analysis of Multivariate Observations\fg{},
  5th Berkeley Symposium on Mathematical Statistics and Probability., p. 281--297, 1967.

  \bibitem[Otsu 1979]{Otsu79}
  N. Otsu,
  \og{}A threshold selection method from gray-level histograms\fg{},
  \emph{ IEEE Transactions on Systems, Man, and Cybernetics} 9(1) p. 62--66, 1979.

  \bibitem[Sezguin et Sankur 2004]{Sezgin04}
  M. Sezgin, B. Sankur,
  \og{}Survey over image thresholding techniques and quantitative performance evaluation\fg{},
  \emph{Journal of Electronic Imaging} 13(1), p. 146--165, 2004.

  \bibitem[Steinhaus 1957]{Steinhaus57}
  H. Steinhaus,
  \og{}Sur la division des corps matériels en parties\fg{}
  \emph{Bulletin de l'Académie Polonaise des Sciences}, 4(12) p. 801--804, 1957.
 -->
