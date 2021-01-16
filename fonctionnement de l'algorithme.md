Première étape :
Transformation du stl en tableau de :
svg (chemins), ou multiples polygones, ou polygone + courbes

A partir de là, la direction de remplissage doit être connue.
Soit on utilise le défaut (des diagonales, perpendiculaires d'une coucuhe à l'autre), soit si donné en option, on utilise un script python à qui on donne divers informations, dont la couche courante, et éventuellement le "polygone(ou la surface fermée/chemin)" courant, et qui renvoie l'angle de remplissage, si on souhaite le personnaliser davantage.

Définition du mode d'overlapping : additif (défaut), booleen, couleur soustractives (pour svg uniquement).
On simplifiera en parlant de polygone pour désigner les chemins, les polygones avec courbes (bézier, arc de cercle), et toutes surfaces (pas le plan).
On vérifie la cohérence du fichier produit.
On résout la logique booléenne pour savoir ce qui est en positif ou en nul. Et on reforme les polygones.
Les polygones doivent être strictement connexe. Ceux qui ne sont connexe que par un point sont scindés. En particulier : toute auto intersectino mène à la scission.
Tout point avec trois ou quatre segments concourants est également point de scission.

Ainsi on obtient des polygone (en réalite mathématiquement des ensembles) qui sont strictement connexe et dont l'""épaisseur" est toujours non nulle.*
A cet effet, on peut mentionner le cas trivial de l'allé retour d'un segment, qui est à supprimer.

Ona obtient donc des polygones. Cependant ceux-ci peuvent parfois avoir le mauvais goût d'être formé de plusieurs chemin (ils peuvent contzenir des trous).
Pour chaque polygone, on relie le polygone à un autre par un allé retour, coupant le chemin de chacun des deux polygone, faisant ainsi qu'il n'y a plus qu'un chemin.
A ce moment, on aurait pu séparer tous les polygones non convexe en sous polygones convexe.
Cependant on utilise une approche qui diminuera la section de surface, afin d'obtenir des surfaces plus grandes :
En gardant la direction de remplissage, et du sens de remplissage (vecteur) on va suivre le chemin dans le sens deds aiguilles d'une montre (on suppose ici que le chemin finit là où il commence, sinon on pourrait avoir des escargots (apr exemple) et ce qui suit ne serait plus valide). On mesure l'angle (fr -pi à pi) que forme le morceau de chemin courant et le sens de remplissage. Ci il est accepté qu'il ne soit pas monotone, celui ne doit cependant jamais repasser deux fois par zéro dans le sens opposé (trois fois en fait), ni par pi. Quand c'est le cas, on prend la dérivé de l'angle des chemins entre les deux intersections, on regarde entre quels sections elle change de signe, et le point enjtre ces deux sections devient le départ d'un nouveau double chemin qui coupe le polkygone, pour le rendre ""psedo convexe"" du point de vue de l'angle de remplissage.

Commence ensuite le traçage. On le fait polygone par polygone. On aura préalablement différencié les frontirèe réelles (entre du positif et du nul, de la matière et du "pas de matière") . On suit les frontières réelles pour faire les périmètres.
On fait ensuite le remplissage.
Le tout se fait dans une boucle multipleùment imbriqué.
Chaque indexe de chaque boucle et surboucle forme le contexte.

Ce contexte et gardé et accessible à chaque instant, car ilm permettra d'ajouter du gcode personnalisé ou des caractéristiques.

En effet, un fichier de contrainte (avec ordre d'application) est fourni. Un moteur le parse (la priorité est à la dernière contrainte en cas de multiple correspondance).
Pour chaque morceau de chemin, la dernière contrainte matchant (avec filtre sur le contexte : polygone courant, surface réelle du sous polygone, n° de tranche, présence ou non de surface sur la tranche du dessous (tous les contextes sont accessibles dans un grand tableeau de tableau de ...)) définit le comportement. Lorsqu'il ne s'agit pas simplement d'un modifieur (quantité d'extrusion), mais d'uun exécuteur, son gcode est ajouté. Lorsqu'il s'agit d'un remplaceur, son gcode qu'il génèrera sera utilisé à la place. Les remplcaeur peuvent aussi altérer le contexte (polygones, etc, et aussi des espaces de contexte sont réservés aux commentaires).

/*

A partir de cette direction
Verification de la cohérence : les polygones inclus sont "discard"
*/




Première étape : la formation des tranches : le stl contient uniquement des triangles, décrivant la surface d'un objet.

On commence par en réaliser la projection.
On fait un tableau de X couches, en fonction de l'épaisseur de couches demandées.
On prend le sommet le plus haut et le sommet le plus bas. Ca nous donne l'intervalle des couches dans lesquelles on va mettre ce triangle.

Ensuite on itère sur les couches. chaque triangle a donc une (projection) intersection (!) non vide sur cette couche.
Pour la calculer, c'est très simple :
On réalise une disjonction de cas :
Les 3 vertex ont un z égal au z de la couche courante (pour chaque couche on ne considère pas l'intrballe mais le z moyen).
Dans ce cas, le triangle (éventuellement plat) est inclus dans le plan. Etant donné que les volumes doivent former un intérieur, chaque côté a forcément un autre triangle adjacent. (De plus, chaque sommet de ce triangle a au total (incluant le triangle en cours de traitement) au minimum 3 triangles qui en partent)
Autrement dit, le triangle sera forcément doublé. Il est donc inutile (voir même nuisible selon la suite de l'algorithme), de le conter.
On passe au triangle suivant
2 vertex ont la propriété ci dessus
Dans ce, la projection est le segment dont les deux vertex sont l'extrémité
1 vertex a la propriété ci dessus
  Nouvelle disjonction de cas : si les deux autres
