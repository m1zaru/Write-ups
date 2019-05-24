**Scully (1) from ECSC 2019**

On commence par voir un indice avant de lancer le challenge !

```
Notre développeur web n'est pas très doué en matière de sécurité... saurez-vous afficher le flag afin que nous puissions lui montrer qu'il faut faire des efforts ?

Le challenge est disponible à l'adresse http://challenges.ecsc-teamfrance.fr:8003

Aucun bruteforce n'est nécessaire

```

"Notre développeur web n'est pas très doué en matière de sécurité", ça sera donc une vulnérabilité simple (selon moi)

On commence par regarder le code de la page principal, et on voit que le formulaire utilise du POST, et fait appel à une fonction nommé "login()", il y a donc un .js externe.

En regardant dans le head, j'ai vu donc un script stocké dans le path "/static/"

<img src="https://i.imgur.com/q0NUWPQ.png" />

En regardant le code JS, on peut voir qu'avec le bon login, il nous donnera le flag.

Mais, la variable "dat" m'a fait tilté qu'on va devoir utilisé une NoSQL Injection.

```
var dat = {'username':username, 'password':password};
```

Essayons donc un auth bypass SQL (http://www.lifeoverpentest.com/2018/03/sql-injection-login-bypass-cheat-sheet.html)

On se connecte avec notre payload et n'importe quel mdp

<img src="https://i.imgur.com/6LCc4VF.png" />

Et voilà, le flag est donc ***ECSC{889b71de2017ca8074f49d3f853950e147591b38}***
