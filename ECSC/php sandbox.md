**PHP sandbox wup from ECSC 2019**

Bon, on avait déjà un bon indice pour commencer: "Trouver les bons arguments pour lui parler !"

On peut vite comprendre qu'un argument sera à passer, testons quelques arguments

J'ai fait une petite liste d'args à essayer:

*Théorie*

```
?cmd=
?args=
?arguments= 
?passthru=
?ev=
?command=
?file=
?dir=
```

Après en avoir essayer quelques uns, j'ai pu constater qu'il n'y en a que un qui est prit en compte: "?args="

[https://github.com/sMizaru/Write-ups/blob/master/ECSC/Images/Capture%20du%202019-05-22%2020-36-44.png]


Déjà, cela parrait logique que cet arguments cat un fichier, et sûrement le fichier que nous avons en argument.
Essayons de cat le fichier flag.txt

[https://github.com/sMizaru/Write-ups/blob/master/ECSC/Images/Capture%20du%202019-05-22%2020-39-54.png]

Une nouvelle erreur apparait qui nous dit clairement que le preg_match() n'accepte pas les caractères que nous lui avons passé precedemment, serait-ce le "." ? Testons de l'enlever.

image

Non, ce n'est pas le ".", serait-ce donc l'espace ? Essayons aussi de l'enlever.

[https://github.com/sMizaru/Write-ups/blob/master/ECSC/Images/Capture%20du%202019-05-22%2020-57-05.png] 

En effet, c'est bien l'espace qui dérange le preg_match(). Essayons donc de bypass le preg_match().

On tombe très vite sur des méthodes (d'ailleurs merci à ceux qui m'ont redirigé sur une bonne méthode)


Essayons donc cette technique

[https://github.com/sMizaru/Write-ups/blob/master/ECSC/Images/Capture%20du%202019-05-22%2020-50-02.png]

On voit donc qu'elle marche avec succès, maintenant, trouvons le fichier flag.txt

Essayons de remonter d'autant de directory qu'il faut pour pouvoir cat le flag.

[https://github.com/sMizaru/Write-ups/blob/master/ECSC/Images/Capture%20du%202019-05-22%2021-01-21.png]

Après réflexion, je n'ai pas penser à regarder dans le home.

[https://github.com/sMizaru/Write-ups/blob/master/ECSC/Images/Capture%20du%202019-05-22%2021-02-21.png]

Voilà nous avons le flag ! 
