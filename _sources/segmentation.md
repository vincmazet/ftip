(C:segmentation)=
# Segmentation

% =============================================================================================== %

## Introduction

La segmentation consiste à partitionner une image $f$ suivant un critère d'homogénéité.


  <!-- \parbox{35mm}{\includegraphics[width=35mm]{marguerite}}
  \parbox{10mm}{\centering$\rightarrow$}
  \parbox{35mm}{\includegraphics[width=35mm]{marguerite-seg}}


* La partition est un ensemble de régions $R_i$ disjointes qui recouvrent l'intégralité de l'image.
* Le critère d'homogénéité est vérifié par chaque région $R_i$
  et n'est pas vérifié pour l'union de deux régions adjacentes.

La {numref}`F:segmentation:examples` XXX présente plusieurs exemples de méthodes de segmentation.

\parbox{25mm}{\includegraphics[width=25mm]{bacteries}\imgref{NASA/JPL-Caltech}}
\parbox{5mm}{\centering$\rightarrow$}
\parbox{25mm}{\includegraphics[width=25mm]{bacteries-seg}}
\parbox{48mm}{%
  Critère :\par niveaux de gris\par\medskip
  Méthode :\par seuillage de l'histogramme
}

\parbox{25mm}{\includegraphics[width=25mm]{marguerite}}
\parbox{5mm}{\centering$\rightarrow$}
\parbox{25mm}{\includegraphics[width=25mm]{marguerite-seg}}
\parbox{48mm}{%
  Critère :\par couleur\par\medskip
  Méthode :\par classification
}

% Kozegar, Soryani , Behnam, Salamati, Tan "Mass Segmentation in Automated 3-D Breast Ultrasound Using Adaptive
% Region Growing and Supervised Edge-Based Deformable Model", IEEE Trans. Medical Imaging, vol. 37, no. 4,2018
\parbox{25mm}{\includegraphics[width=25mm]{breast}\imgref{Kozegar 2018}}
\parbox{5mm}{\centering$\rightarrow$}
\parbox{25mm}{\includegraphics[width=25mm]{breast-seg}}
\parbox{48mm}{%
  Critère :\par différence d'intensité\par\medskip
  Méthode :\par croissance de région
}

\parbox{25mm}{\includegraphics[width=25mm]{haiti}\imgref{ICube/Sertit}}
\parbox{5mm}{\centering$\rightarrow$}
\parbox{25mm}{\includegraphics[width=25mm]{haiti-seg}}
\parbox{48mm}{%
  Critère :\par couleur + taille des régions\par\medskip
  Méthode :\par SLIC \mycite{Achanta12}
}

% Autres exemples de méthodes de segmentation
%
% \parbox{25mm}{\includegraphics[width=25mm]{astronaut}}
% \parbox{5mm}{\centering$\rightarrow$}
% \parbox{25mm}{\includegraphics[width=25mm]{astronaut-seg}}
% \parbox{48mm}{%
%   Critère :\par contours\par\medskip
%   Méthode :\par contours actifs
% }
%
% \parbox{25mm}{\includegraphics[width=25mm]{moon}\imgref{NASA}}
% \parbox{5mm}{\centering$\rightarrow$}
% \parbox{25mm}{\includegraphics[width=25mm]{moon-seg}}
% \parbox{48mm}{%
%   Critère :\par contours\par\medskip
%   Méthode :\par ligne de partage des eaux
% }
%
% Télédétection : classification d'une région agricole[\baselineskip]
% \includegraphics[width=\textwidth]{INRIA-CDR0001-0043_HD}
% \imgref{Inria / Ariana}
%
% imagerie médicale : planification opératoire[\baselineskip]
% \includegraphics[width=11cm]{segmentation3D}
%
% Vidéo : incrustation, effets spéciaux, ...[\baselineskip]
% \includegraphics[width=\textwidth]{incrustation_reussie}
%
% \includegraphics[width=5cm]{zebre}
% Texture

% Article de revue : \mycite{Sezgin04} (webdocs.cs.ualberta.ca/ nray1/CMPUT605/track3_papers/Threshold_survey.pdf)
% We categorize the thresholding methods in six groups according to the information they are exploiting:
% 1. histogram shape-based methods (eg the peaks, valleys and curvatures of the smoothed histogram are analyzed)
% 2. clustering-based methods (the gray-level samples are clustered in two parts or alternately are modeled as a mixture of two Gaussians)
% 3. entropy-based methods (entropy of the foreground and background regions, cross-entropy between original and binarized image, etc.)
% 4. object attribute-based methods (search a measure of similarity between the gray-level and the binarized images)
% 5. the spatial methods (higher-order probability distribution and/or correlation between pixels)
% 6. local methods (adapt the threshold value on each pixel to the local image characteristics)

Ce chapitre présente dans plusieurs méthodes de segmentation,
et termine sur la façon de mesurer la performance d'une méthode de segmentation.
Mais avant tout, il est utile de définir quelques termes liés à la relation qui peut exister entre les pixels.

% cf aussi GW section 2.5


**Voisinage**

4-voisinage \includegraphics[scale=.8]{voisinage4}
8-voisinage \includegraphics[scale=.8]{voisinage8}
Les pixels en gris sont les voisins du pixel $(m,n)$.

**Composante connexe**

Une composante connexe \eng{connected component} est un groupe de pixels
tel qu'on puisse aller d'un pixel de ce groupe à un autre pixel de ce groupe
en passant par des pixels du même groupe voisins entre eux.

\includegraphics[width=35mm]{connectedcomponent}
\includegraphics[width=35mm]{connectedcomponent4}
\includegraphics[width=35mm]{connectedcomponent8}
Combien de composantes connexes ?
5 composantes connexes en 4-voisinage.
4 composantes connexes en 8-voisinage.

**Remarques**

* Chaque région $R_i$ est une composante connexe.

* Le résultat de la segmentation n'est pas unique
  (dépend du critère d'homogénéité, de la méthode, de l'initialisation, etc.).

* Une segmentation peut s'interpréter comme un graphe\par
  (n\oe{}uds = régions, liens entre régions voisines)
  \includegraphics[width=25mm]{connectedcomponent4}
  \includegraphics[width=25mm]{connectedcomponent4-graph}

* La segmentation donne une représentation haut niveau de l'image. -->

% =============================================================================================== %

## Histogram thresholding

<!-- % Seuillage (manuel) - dans la suite : différentes méthodes de seuillage auto
% seuillage de Otsu
% seuillage multiple
% utilisation des contours
% k-moyennes \todo{pour PC} (évoquer \emph{mean-shift}, modèles paramétriques), SLIC
% influence de l'éclairage
% influence du bruit


Distinction pixels clairs et foncés $\rightarrow$ binarisation de l'image

  \parbox{35mm}{\includegraphics[width=35mm]{bacteries}\imgref{NASA/JPL-Caltech}\par\medskip\centering $f$}
  \qquad
  \parbox{35mm}{\includegraphics[width=35mm]{bacteries-seg}\par\medskip\centering $g$}

$$
  g(m,n) =
  \begin{cases}
    1 & \text{si $f(m,n)\geqslant T$}
    0 & \text{si $f(m,n)< T$}
  \end{cases}
  \qquad\text{où $T$ est le seuil}
$$

Comment choisir le seuil $T$ ?

  \includegraphics[width=.32\textwidth]{imageNB}
  \includegraphics[width=.32\textwidth]{seuil70}
  \includegraphics[width=.32\textwidth]{seuil150}\\[.5ex]
  \parbox[t]{.32\textwidth}{\centering Image originale (256 niveaux de gris sur $\{0,...,255\}$)}
  \parbox[t]{.32\textwidth}{\centering Seuil à 70}
  \parbox[t]{.32\textwidth}{\centering Seuil à 150}

Un outil intéressant pour le choix du seuil : l'histogramme.

Dans certains cas, le choix du seuil est facile :

  \includegraphics[height=32mm]{finger}
  \includegraphics[height=32mm]{finger-hist-125}
  \includegraphics[height=32mm]{finger-seg-125}

Dans d'autres cas, le choix du seuil est moins évident :

  \includegraphics[height=32mm]{spider}
  \includegraphics[height=32mm]{spider-hist-195}
  \includegraphics[height=32mm]{spider-seg-195}

  \includegraphics[height=32mm]{spider}
  \includegraphics[height=32mm]{spider-hist-100}
  \includegraphics[height=32mm]{spider-seg-100}

### Méthode de Otsu

La méthode de Otsu \mycite{Otsu79} permet de déterminer le seuil
qui minimise la \emph{variance intra-classe} $\sigma_w^2(T)$ (moyenne pondérée des variances de chaque classe) :
\begin{align*}
  T &= \arg \min_T \sigma_w^2(T)
    &= \arg \min_T q_1(T)\sigma_1^2(T) + q_2(T)\sigma_2^2(T)
\end{align*}

  \includegraphics[width=40mm]{celine20}

$$
  T = \arg \min_T q_1(T)\sigma_1^2(T) + q_2(T)\sigma_2^2(T)
$$

Pour la classe 1 :
\begin{tabular}{lc}
  Proportion :    & $\displaystyle q_1(T) = \sum_{i = 0}^{T} p(i)$                                         
  Moyenne :       & $\displaystyle m_1(T) = \frac{1}{q_1(T)} \sum_{i = 0}^{T} i p(i)$                      
  Variance :      & $\displaystyle \sigma^2_1(T) = \frac{1}{q_1(T)} \sum_{i = 0}^{T}(i\!-\!m_1(T))^2 p(i)$
\end{tabular}

Les intensités $i$ sont à valeurs dans $\{0,...,L-1\}$,
$p(i) = h(i)/(M \times N)$ est l'histogramme normalisé
et $h$ l'histogramme.

Pour la classe 2 :
\begin{tabular}{lc}
  Proportion :    & $\displaystyle q_2(T) = \sum_{i = T+1}^{L-1} p(i)$                                         
  Moyenne :       & $\displaystyle m_2(T) = \frac{1}{q_2(T)} \sum_{i = T+1}^{L-1} i p(i)$                      
  Variance :      & $\displaystyle \sigma^2_2(T) = \frac{1}{q_2(T)} \sum_{i = T+1}^{L-1}(i\!-\!m_2(T))^2 p(i)$
\end{tabular}

Les intensités $i$ sont à valeurs dans $\{0,...,L-1\}$,
$p(i) = h(i)/(M \times N)$ est l'histogramme normalisé
et $h$ l'histogramme.

% TODO : exemples d'images seuillées avec Otsu

Implémentation :
Calculer pour tous les seuils $T = \{0,...,L-1\}$
la variance intra-classe $\sigma_w^2(T)$,
et retenir le seuil $T$ qui minimise $\sigma_w^2(T)$.

 \setlength{\fboxsep}{3mm}
 \colorbox{algobg}{\parbox{.9\textwidth}{
 Pour $T = 0 \rightarrow L-1$ :\\
 \albar Calculer $\sigma_w^2(T)$
 Choisir $\displaystyle\hat{T} = \arg\min_T \sigma_w^2(T)$
 }}

Remarque :
La variance $\sigma^2$ des intensités de l'image s'écrit :

$$
  \sigma^2 = \sigma_w^2 + \sigma_{1,2}^2
$$
où $\sigma_{1,2}^2$ est la \emph{variance inter-classe} (variance pondérée des moyennes de chaque classe).

$\Rightarrow$\quad Minimiser la variance intra-classe $\sigma_w^2$
est équivalent à maximiser la variance inter classe $\sigma_{1,2}^2$
(puisque $\sigma^2$ reste constant).

$\Rightarrow$\quad Construire deux groupes de \og{}pixels qui se ressemblent\fg{}
revient à construire deux \og{}groupes très dissemblables\fg{} de pixels.

La méthode de Otsu nécessite le réglage d'un paramètre par l'utilisateur : vrai ou faux ?
  \only<beamer>{$\rightarrow$ Faux}

La méthode de Otsu tend à avoir des intensités, au sein d'une classe, les plus proches possibles : vrai ou faux ?
  \only<beamer>{$\rightarrow$ Vrai}

La méthode de Otsu tend à avoir des intensités entre deux classes les plus proches possibles : vrai ou faux ?
  \only<beamer>{$\rightarrow$ Faux}

### Seuillage multiple

Lorsque plusieurs modes sont visibles sur l'histogramme,
il est possible d'utiliser plusieurs seuils pour aboutir à plusieurs classes :

  \includegraphics[height=32mm]{brain}
  \includegraphics[height=32mm]{brain-hist}
  \includegraphics[height=32mm]{brain-seg}

En particulier, la méthode de Otsu peut être étendue à plusieurs seuils,
mais la complexité calculatoire augmente grandement avec le nombre de classes !

% La variation d'illumination ne permet pas de seuiller correctement l'image. Plusieurs solutions sont possibles.
%
% TODO : afficher :
% - image originale + son histogramme + son seuillage
% - fond
% - image sans fond + son histogramme + son seuillage
%
%   \includegraphics[width=5cm]{seuillage_variation_illumination}
%   \rput(-5.3,5.3){$f$}
%   \rput(-5.3,3.4){$g$}
%   \rput(-5.3,0.9){$h$}
%   \imgref{Gonzalez $\&$ Woods}
%
% Éclairage $g$ connu : on utilise un modèle paramétrique pour le décrire et on corrige l'image avant le seuillage :
%   $$
%     \forall (m,n), \quad  h(m,n) = \frac{f(m,n)}{g(m,n)}
%   $$
%
% * Éclairage $g$ inconnu : seuillage local par exemple
%   \includegraphics[width=8.5cm]{seuillage_bloc}
%   \imgref{Gonzalez $\&$ Woods}
% \pnode(3.8,1.2){A}
% \rput[lt](2,0.3){\rnode{B}Attention : zone à une seule classe ($\Rightarrow$ test de la variance)}
% \ncline[linecolor=gray]{<-}{A}{B}
%
% \begin{frame}{Influence du bruit}
%
% Ajout d'un bruit gaussien sur l'image $\Rightarrow$ convolution de l'histogramme de l'image par une gaussienne.
%   \includegraphics[width=10cm]{seuillage_bruit}
%   \imgref{Gonzalez $\&$ Woods}
%
% Solutions possibles :
% * Débruiter l'image initiale (filtre gaussien, filtre moyenneur, méthode de débruitage, ...)
% * Filtrer l'image seuillée (opérateurs morphologiques, filtre médian, ...)
%   \only<2>{\rput[c](.5\linewidth,-2){\includegraphics[width=8cm]{vincent72}}}
% * Incorporer de l'information spatiale dans la méthode de segmentation.
%   \includegraphics[width=10cm]{seuillage_filtre_median} -->

% =============================================================================================== %

## Classification

<!-- % k-moyennes \todo{pour PC} (évoquer \emph{mean-shift}, modèles paramétriques), SLIC
% influence de l'éclairage
% influence du bruit

Comment segmenter une image multibande ?
Par exemple, une image RVB possède trois bandes, pour chacune desquelles peut être déterminée un histogramme.

\includegraphics[height=3cm]{imgclr}

$\Rightarrow$ segmentation de l'image à l'aide d'un histogramme impossible !

Un pixel est maintenant représenté par un vecteur à valeurs dans $\mathbb{R}^B$
$\Rightarrow$
travailler directement dans l'espace $\mathbb{R}^B$ !

  \includegraphics[height=3cm]{imgclr}\hspace*{2cm}
  \includegraphics[height=3cm]{celine29}

Le principe des méthodes de classification (ou plus exactement de coalescence) \eng{clustering}
est de regrouper les pixels en groupes homogènes.

* Algorithme des k-moyennes \mycite{Steinhaus57,MacQueen67}
* Modèles paramétriques (mélange de lois)
* \emph{Mean-shift} \mycite{Fukunaga75}
* SLIC \mycite{Achanta12}
* ...

### Algorithme des k-moyennes

L'algorithme des k-moyennes \eng{k-means}
est une méthode itérative qui affecte chaque point de l'espace $\mathbb{R}^B$
à l'un des $K$ groupes \eng{clusters} ($K$ choisi par l'utilisateur).

 \setlength{\fboxsep}{3mm}
 \colorbox{algobg}{\parbox{.9\textwidth}{
  Initialisation aléatoire de $K$ centroïdes
  Répéter tant que les centroïdes varient :
  \albar Pour chaque point :
  \albar\albar Calcul des distances du point à tous les centroïdes
  \albar\albar Affectation du point au groupe le plus proche
  \albar Calcul du centroïde de chacun des groupes
 }}

Exemple
Pour simplifier, on considère une image à deux bandes ($\Rightarrow$ espace à deux dimensions)
à segmenter en $K=2$ classes ($\Rightarrow$ deux couleurs).

  {\color<2>{darkgray}Initialisation des centroïdes}
  {Tant que les centroïdes varient :}
  \albar {Pour chaque point :}
  \albar\albar {\color<3,5,7>{darkgray}Distances aux centroïdes}
  \albar\albar {\color<3,5,7>{darkgray}Affectation}
  \albar {\color<4,6,8>{darkgray}Mise à jour des centroïdes}

  \includegraphics[width=25mm]{kmeans0}
  \includegraphics[width=25mm]{kmeans1}
  \includegraphics[width=25mm]{kmeans2}
  \includegraphics[width=25mm]{kmeans3}\\
  \includegraphics[width=25mm]{kmeans4}
  \includegraphics[width=25mm]{kmeans5}
  \includegraphics[width=25mm]{kmeans6}
  \includegraphics[width=25mm]{kmeans7}

\imgbox{35mm}{marguerite}{Image originale}
\imgbox{35mm}{marguerite-kmeans-2}{$K=2$}
\imgbox{35mm}{marguerite-kmeans-4}{$K=4$}

Avantages
* Méthode simple
* Implémentation facile
* Méthode généralement rapide
* Classes de variance conditionnelle minimale
* Fonctionne correctement lorsque les clusters sont sphériques
  \includegraphics[height=12mm]{vinc51-1}

Inconvénients
* Nécessite de connaître le nombre de classes
* Sensible aux minima locaux, donc à l'initialisation
* Peut être lent en grande dimension
* Échoue pour des structures non sphériques
  \includegraphics[height=14mm]{vinc51-2}
* Sensible aux valeurs aberrantes
  \includegraphics[height=14mm]{vinc51-3}

### Modèles paramétriques

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

<!-- % Croissance de région
% Segmentation par décomposition et regroupement / décomposition et fusion

### Limitation des méthodes de seuillage

La limite fondamentale des méthodes de seuillage est
de ne pas prendre en compte l'information de voisinage
(seule l'information de distribution des intensités est utilisée).

L'avantage des méthodes basées région est d'agréger des pixels spatialement proches
_et_ ayant des intensités similaires.

* Croissance de région
* Décomposition / fusion

### Croissance de région (_Region growing_)

Principe :
on part d'un pixel initial (\og{}germe\fg{})
et on l'étend en ajoutant les pixels du voisinage
satisfaisant le critère d'homogénéité.

\parbox[t]{18mm}{\includegraphics[width=18mm]{marguerite-region-growing-seed}}
\raisebox{8mm}{$\rightarrow$}
\parbox[t]{80mm}{%
  \includegraphics[width=18mm]{marguerite-region-growing-1}
  \includegraphics[width=18mm]{marguerite-region-growing-2}
  \includegraphics[width=18mm]{marguerite-region-growing-3}
  \includegraphics[width=18mm]{marguerite-region-growing-4}\\[2mm]
  \includegraphics[width=18mm]{marguerite-region-growing-5}
  \includegraphics[width=18mm]{marguerite-region-growing-6}
  \includegraphics[width=18mm]{marguerite-region-growing-7}
  \includegraphics[width=18mm]{marguerite-region-growing-8}
}

Choix du germe :
* manuellement (dans la zone d'intérêt)
* automatiquement (en évitant les zones de fort contraste $=$ fort gradient)

Critère de similarité :
si un pixel $f(m,n)$ et une région $R$ sont suffisamment similaires,
alors ils sont fusionnés ;
sinon une nouvelle région est créée.

Exemple de critère :
$$
  \left| f(m,n) - \mu_R \right| < T \sigma_R
$$

* $T$ élevé : facile d'agréger des nouveaux pixels à la région.
* $T$ faible : difficile d'agréger des nouveaux pixels à la région.

Choix de la connexité : 4-voisinage ou 8-voisinage.

* $R$ : région segmentée, initialisée au germe
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

La croissance de région ne fournit pas directement une partition de l'image,
mais permet de segmenter une ou plusieurs structures d'intérêt via la sélection de germes adaptés.

Au moins deux points germes sont nécessaires :

  \imgbox{45mm}{eclairs}{Image originale}\qquad
  \imgbox{45mm}{eclair1}{Image segmentée}

Quelle segmentation est obtenue avec la plus grande valeur de $T$ ?

  \only<handout>{%
    \imgbox{50mm}{eclair1}{A}\qquad
    \imgbox{50mm}{eclair3}{B}%
  }
  \only<beamer>{%
    \imgbox{40mm}{eclair1}{\only<1>{A}\only<2>{$T$ petit}\phantom{gT}}\qquad
    \imgbox{40mm}{eclair3}{\only<1>{B}\only<2>{$T$ grand}\phantom{gT}}%
  }

% TODO : ??? \textcolor{red!70}{Rq : en cas d'utilisation de statistique globale pour le test d'homogénéité, l'ordre de traitement des pixels peut influencer le résultat final.}

### Décomposition / fusion (_Split and merge_)

Principe :
1. Divisions \eng{split} successives de chaque région de l'image
  si elles ne satisfont pas le critère d'homogénéité
  $\Rightarrow$ Permet d'aboutir à une \og{}partition initiale\fg{}.
  \onslide<2->{$\rightarrow$ représentation par arbre.}

1. Fusions \eng{merge} successives des régions adjacentes satisfaisant le critère d'homogénéité.
  \onslide<3->{$\rightarrow$ représentation par graphe d'adjacence.}

\onslide<4->{Les représentations en arbre et par graphe permettent une représentation haut niveau de l'image.}

Le quad-arbre (_quad-tree_) est une arborescence dont chaque nœud représente une région et possède quatre fils
(la racine représente l'image entière).

  \includegraphics[height=5cm]{quadarbre-ex}

**Décomposition**

Chaque région $R$ est partitionnée en quatre régions de taille identique
si elle ne respecte pas le critère d'homogénéité.

\begin{minipage}{3cm}
  Représentation par quad-arbre :
\end{minipage}
\begin{minipage}{7cm}\centering
  %\only<1>{\includegraphics[height=5cm]{../../com/blank}}
  \only<2>{\includegraphics[height=5cm]{quadarbre1}}%
  \only<3>{\includegraphics[height=5cm]{quadarbre2}}%
  \only<4>{\includegraphics[height=5cm]{quadarbre3}}%
  \only<5>{\includegraphics[height=5cm]{quadarbre4}}%
\end{minipage}

Chaque région $R$ est partitionnée en quatre régions de taille identique
si elle ne respecte pas le critère d'homogénéité.

  \includegraphics[width=25mm]{quadarbre1}%
  \includegraphics[width=25mm]{quadarbre2}%
  \includegraphics[width=25mm]{quadarbre3}%
  \includegraphics[width=25mm]{quadarbre4}%

* La méthode de décomposition par quad-arbre fait apparaître des régions carrées sur l'image segmentée.
* Le problème majeur de cette structure provient de la rigidité des divisions réalisées sur l'image, mais cela fournit une partition initiale de l'image.

Le <span title="RAG : region adjacency graph" style="text-decoration:dashed;color:red;">graphe d'adjacence</span> (_RAG : region adjacency graph_) est un graphe dont :

* les nœuds correspondent à une région de l'image,
* les arêtes relient les nœuds correspondants à deux régions adjacentes (ayant une frontière commune).

  \includegraphics[width=10cm]{vincent88}

**Fusion**

À partir d'une partition de l'image (par exemple obtenue avec un quad-arbre),
on fusionne les n\oe{}uds $R_1$ et $R_2$ voisins et dont le critère de similarité sur $R_1 \cup R_2$
est respecté.

\begin{minipage}{3cm}
  Représentation par un graphe d'adjacence :
\end{minipage}
\begin{minipage}{7cm}\centering
  % \only<1>{\includegraphics[height=5cm]{../../com/blank}}
  \only<2>{\includegraphics[height=5cm]{rag3}}%
  \only<3>{\includegraphics[height=5cm]{rag4}}%
  \only<4>{\includegraphics[height=5cm]{rag5}}%
  \only<5>{\includegraphics[height=5cm]{rag6}}%
\end{minipage}

À partir d'une partition de l'image (par exemple obtenue avec un quad-arbre),
on fusionne les n\oe{}uds $R_1$ et $R_2$ voisins et dont le critère de similarité sur $R_1 \cup R_2$
est respecté.

  \includegraphics[width=27mm]{rag3}%
  \includegraphics[width=27mm]{rag4}%
  \includegraphics[width=27mm]{rag5}%
  \includegraphics[width=27mm]{rag6}%

**Résumé**

1. Partition initiale en fonction du critère d'homogénéité (par exemple avec un quad-arbre)
1. Fusion des zones segmentées adjacentes en fonction du critère d'homogénéité
  (représentation avec un graphe d'adjacence)

\includegraphics[width=8cm]{celine46} -->

% =============================================================================================== %

## Watershed

<!-- Ligne de partage des eaux \eng{watershed}

L'image est vue comme une carte d'élévation (terrain 3D) où :
* les régions sont les vallées
* les frontières entre régions sont les crêtes

$\rightarrow$ utilisation de la norme du gradient de l'image.

  \imgbox{30mm}{Lune}{Image originale} \qquad
  \imgbox{30mm}{Lune_gradient}{Gradient de l'image} \qquad
  \imgbox{30mm}{Lune_gradient_3D}{Représentation 3D du gradient}

Principe :
* Construire la carte d'élévation
* Remplir progressivement d'eau chaque \textit{bassin versant}
* Lorsque l'eau monte et que deux bassins se rejoignent, la ligne de partage des eaux est marquée comme frontière

Illustration sur un signal 1D :
  \includegraphics<1>[height=3.5cm]{celine49-1}%
  \includegraphics<2>[height=3.5cm]{celine49-2}%
  \includegraphics<3>[height=3.5cm]{celine49-3}%
  \includegraphics<4>[height=3.5cm]{celine49-4}%
  \includegraphics<5>[height=3.5cm]{celine49-5}%
  \includegraphics<6>[height=3.5cm]{celine49-6}%
  \includegraphics<7>[height=3.5cm]{celine49-7}%
  \includegraphics<8>[height=3.5cm]{celine49-8}%
  \includegraphics<9>[height=3.5cm]{celine49-9}%

Principe :
* Construire la carte d'élévation
* Remplir progressivement d'eau chaque \textit{bassin versant}
* Lorsque l'eau monte et que deux bassins se rejoignent, la ligne de partage des eaux est marquée comme frontière

Illustration sur un signal 1D :
  \includegraphics[height=3.5cm]{celine49-6}

Algorithme :
 \setlength{\fboxsep}{3mm}
 \colorbox{algobg}{\parbox{.9\textwidth}{
  Calculer le gradient (ou le Laplacien) de l'image.
  Les pixels ayant l'intensité la plus faible forment les bassins \phantom{\albar}\quad versants initiaux.
  Pour chaque niveau d'intensité $i$ :
  \albar Pour chaque groupe de pixels d'intensité $i$ :\\
  \albar\albar Si adjacent à exactement une région existante :\\\albar\albar\quad ajouter ces pixels dans cette région.\\
  \albar\albar Si adjacent à plusieurs régions simultanément :\\\albar\albar\quad marquer comme ligne de partage des eaux.
  \albar\albar Sinon, commencer une nouvelle région.

% * Calculer le gradient (ou le Laplacien) de l'image.
% * Les pixels ayant l'intensité la plus faible forment les bassins versants initiaux.
% * Pour chaque niveau d'intensité $i$ :
%   * Pour chaque groupe de pixels d'intensité $i$ :
%     * Si adjacent à exactement une région existante : ajouter ces pixels dans cette région.
%     * Si adjacent à plusieurs régions simultanément : marquer comme ligne de partage des eaux.
%     * Sinon, commencer une nouvelle région.

Limitation
Sur-segmentation s'il y a beaucoup de minima locaux dans le gradient.

  \imgbox{3.5cm}{vincent103-1}{Image}\qquad
  \imgbox{3.5cm}{vincent103-2}{Gradient}\\[1ex]
  \imgbox{3.5cm}{vincent103-3}{LPE}\qquad
  \parbox{3.5cm}{ }

% TODO : l'exemple (fig 3) est trop complexe et difficle à lire : simplifier

Solutions possibles :
* lisser (filtrage passe-bas) le gradient avant d'appliquer l'algorithme
* choisir manuellement les bassins versants d'intérêt avec des marqueurs
* fusionner les minima locaux

  \imgbox{3.5cm}{vincent103-1}{Image}\qquad
  \imgbox{3.5cm}{vincent103-2}{Gradient}\\[1ex]
  \imgbox{3.5cm}{vincent103-3}{LPE}\qquad
  \imgbox{3.5cm}{vincent103-4}{\mbox{Utilisation de marqueurs}} -->

% =============================================================================================== %

## Snakes

<!-- Contours actifs

Principe : à partir d'un contour initial proche de l'objet à segmenter,
le contour évolue de manière itérative et cherche à converger
vers les zones de fort gradient ($=$ contour) sous certaines contraintes (forme, longueur, etc.).

Le contour est modélisé par un ensemble de points $(x_i,y_i)$
qui se déplacent légèrement à chaque itération pour déformer le contour.

  \includegraphics[width=7cm]{vincent117-1}

Le contour cherche à minimiser une énergie (ou fonction coût)
qui mesure la qualité de la segmentation :
$$
  E_\text{totale} = E_\text{interne} + \lambda E_\text{externe}
$$

* Énergie interne : encourage certaines configurations de forme
  (régularité, élasticité, a priori de forme, ...)
* Énergie externe : encourage le modèle à converger vers les contours des objets
  (zones de fort gradient)

% Différents types d'énergie interne :
%   \includegraphics[width=8cm]{vincent115} -->

% =============================================================================================== %

## How to evaluate the segmentation?

<!-- \chapter{Évaluation}

Critères d'évaluation de la segmentation

\begin{minipage}{3.4cm}
  \centering
  \includegraphics[width=3.5cm]{eval-verite}\par
  Vérité terrain $f\,^*$\par
  \includegraphics[width=3.5cm]{eval-seg}\par
  Segmentation $f$
\end{minipage}

\begin{minipage}{6cm}
  \centering
  \includegraphics[width=6cm]{eval-zones}\par\medskip
  \begin{tabular}{ll}
    VP : vrai positif &
    VN : vrai négatif
    FP : faux positif &
    FN : faux négatif
  \end{tabular}
\end{minipage}

* Sensibilité $\displaystyle = \frac{\text{VP}}{\text{VP}+\text{FN}}$
* Spécificité $\displaystyle = \frac{\text{VN}}{\text{VN}+\text{FP}}$
* Dice        $\displaystyle = \frac{2\,\text{VP}}{2\,\text{VP}+\text{FP}+\text{FN}} = \frac{2\,|f\, \cap f\,^*|}{|f\,| + |f\,^*|}$
* Jaccard     $\displaystyle = \frac{\text{VP}}{\text{VP}+\text{FP}+\text{FN}} = \frac{|f\, \cap f\,^*|}{|f\, \cup f\,^*|}$ -->

% =============================================================================================== %

## Conclusion

<!-- * La segmentation consiste à diviser l'image en plusieurs régions homogènes.
* L'homogénéité d'une région est basée sur la couleur, la texture, les contours...
* Les méthodes de segmentation sont très diverses : classification, croissance de région,
  décomposition/fusion, contours actifs, ensembles de niveaux \eng{level sets}, ligne de partage des eaux,
  entropie, modèles markoviens...

Bibliographie

  \bibitem[Achanta  et coll. 2012]{Achanta12}
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

* Opérateurs connexes (cf https://perso.esiee.fr/ perretb/I5FM/TAI/connexity/index.html 🇫🇷):
  - Adjacence
  - Composantes connexes
  - Labélisation -->
