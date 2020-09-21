# NOTES


## General Remarks

* Revoir la cohérence du design et de la mise en page sur tout le book.

* Pour chaque chapitre, écrire une intro (sommaire de ce qui va être vu) et une outro (résumé).


# Preface

* Il faut la re-rédiger, elle pas toute petite.


## Histogram

* Rajouter l'exemple simple d'histogramme sur une image 4x4.

* Normalisation de l'histogramme : je n'indique pas dans la forume la taille des bins : il faudrait l'ajouter (il ne sont pas toujours à 1).


## Convolution

* La figure représentant trois exemples de convolution n'est pas très jolie.

## Compression

* Afficher en sur-impression les images des coefficients de la DCT sur la grille des coefficients.

* Afficher l'image de différence en plus de l'originale et de la transformée.


## Mathematical Morphology

* Les exemples d'applications sur la Grèce pourraient être tous ensemble, afin de comparer érosion, dilatation avec ouverture et fermeture ?

* Aller plus loin dans les notions vues, avec extension en niveau de gris (cf dans Jahne) ou présentation des fonctions supplémentaires disponibles dans scikit-image. Inclure par exemple : top-hat filter, transformée de distance (dans Jahne ?), épaissisement (thickening), amainsissement (thinning), squeletisation (skeletons), taillage (pruning), représentation de l'image sous forme d'un arbre...

* Dans le Jahne : quelques exos et interactivités

* J'ai supprimé les deux applications sur la transformée tout-ou-rien (détection de pixels isolés ou de coins). Je ne pense pas que ce soit des choses vraiment importantes.

* Pour faire des images avec quadrillage : faire une fonction qui prend en entrée le array et s'occupe de tout le reste, en remplacement de imshow.
