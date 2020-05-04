<h1>FCSC 2020 - Clepsydre</h1>

<br><br>
<img src="https://img.onii.wtf/i/8t2my.png">

<br>

Connectons-nous au challenge: 
```
mizaru@DESKTOP-23T8VRF:~$ nc challenges2.france-cybersecurity-challenge.fr 6006                                                                                         
[Citation du jour] : "Tout vient à point à qui sait attendre".                                                                                                                                                                                                                                                                                  Entrez votre mot de passe : 
```

Il attend un mot de passe, avec une citation "Tout vient à point à qui sait attendre", on peut tout de suite faire le lien avec une timing attack. Essayons de mettre ça en oeuvre.

<br><br>

Tout d'abord, il faut savoir que si nous entrons un bon char, il va mettre une seconde de plus à nous retourner le message d'erreur, et ça reste quand même notable, ce qui va nous permettre de leak le password.

(J'ai tout essayé à la main, par flemme de script)

<br>

Ce qui donne :

Premier caractère : T
<br>
Deuxième caractère : 3
<br>
Troisième caractère : m
<br>
Quatrième caratère : p
<br>
Cinquième caractère #
<br>
Sixième caractère : !
<br>
<br>

MDP : T3mp#!

```
Entrez votre mot de passe : T3mp#!                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      
Félicitations vous avez su vaincre votre impatience :                                                                                                                                                                                                                                                                                           
FCSC{6bdd5f185a5fda5ae37245d355f757eb0bbe888eea004cda16cf79b2c0d60d32}
```

