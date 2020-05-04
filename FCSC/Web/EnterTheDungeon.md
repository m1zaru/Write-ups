<center><h1>FCSC 2020 - EnterTheDungeon Writeup</h1></center>

<br>
<br>

<img src="https://img.onii.wtf/i/mkegy.png">

On nous demande simplement de trouver le flag. Très bien, connectons-nous au challenge.

En arrivant on tombe sur un beau message : 

<img src="https://img.onii.wtf/i/dljkn.png">

<br>

```
Les admins ont reçu un message indiquant qu'un pirate avait retrouvé le secret en contournant la sécurité du site.
Pour cette raison, le formulaire de vérification n'est plus fonctionnel tant que le développeur n'aura pas corrigé la vulnérabilité
```

Et un peu plus bas un formulaire nous demandant d'entrer le secret pour prouver que nous sommes le dungeon_master: 

<img src="https://img.onii.wtf/i/ym8xs.png">

Regardons maintenant le code source de la page, voir si celui-ci peut nous apporter quelques précisions ! 
<br>
Et, on tombe sur un petit commentaire dans le code source:
```
<!-- Pour les admins : si vous pouvez valider les changements que j'ai fait dans la page "check_secret.php", le code est accessible sur le fichier "check_secret.txt" -->
```

Vraiment pas malin...
<br>

Très bien regardons voir ce fameux fichier check_secret.txt, voici son contenu :

```php


<?php
	session_start();
	$_SESSION['dungeon_master'] = 0;
?>
<html>
<head>
	<title>Enter The Dungeon</title>
</head>
<body style="background-color:#3CB371;">
<center><h1>Enter The Dungeon</h1></center>
<?php
	echo '<div style="font-size:85%;color:purple">For security reason, secret check is disable !</div><br />';
	echo '<pre>'.chr(10);
	include('./ecsc.txt');
	echo chr(10).'</pre>';

	// authentication is replaced by an impossible test
	//if(md5($_GET['secret']) == "a5de2c87ba651432365a5efd928ee8f2")
	if(md5($_GET['secret']) == $_GET['secret'])
	{
		$_SESSION['dungeon_master'] = 1;
		echo "Secret is correct, welcome Master ! You can now enter the dungeon";
		
	}
	else
	{
		echo "Wrong secret !";
	}
?>
</body></html>
```

Et tout de suite, une condition tape à l'oeil, laquelle ? Celle-ci:

```php

if(md5($_GET['secret']) == $_GET['secret'])
	{
		$_SESSION['dungeon_master'] = 1;
		echo "Secret is correct, welcome Master ! You can now enter the dungeon";
		
	}
	else
	{
		echo "Wrong secret !";
    }
```

C'est un bloc conditionnel normal, le plus frappant est la condition:
```php
md5($_GET['secret']) == $_GET['secret']
```

Comment est-ce possible de remplir la condition me diras-tu, ne t'en fais pas jeune padawan, je vais t'expliquer.
L'erreur que le développeur a fait ici est d'utiliser une comparaison dite "loose" (les "==" quoi), au lieu d'une comparaison "strict", ce qui nous laisse place à une belle issue Type Juggling, exploitons-la !

<br>

PS: Je me suis basé sur un WriteUp déjà fait : https://github.com/bl4de/ctf/blob/master/2017/HackDatKiwi_CTF_2017/md5games1/md5games1.md

<br>

Dans ce Writeup on y trouve donc une chaîne qui peut faire l'affaire : 0e215962017
<br>

Essayons ! 

<img src="https://img.onii.wtf/i/hbgil.png">

Et voilà, mais, où est le flag ? Retournons sur l'accueil voir:

<img src="https://img.onii.wtf/i/yvg0s.png">

Damn c'est Mizaru Inshape. Flagged ^-^
