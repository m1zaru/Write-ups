<center><h1>FCSC 2020 - Le Rat Conteur Writeup</h1></center>
<br>
<br>
<br>

<img src="https://img.onii.wtf/i/inglz.png">

<br>

Très bien, nous nous retrouvons avec un fichier "flag.jpg.enc", et nous avons aussi deux hints à notre disposition.

Nous avons déjà dans l'énoncé, l'algorithme et le mode dans lequel il a été chiffré, ainsi que sa clé et la précision que son IV est nul.

<br>
<br>

Voici ce que nous disent les deux hints:

<img src="https://img.onii.wtf/i/p40c2.png">
<br>
<img src="https://img.onii.wtf/i/7qjzx.png">

<br>

Nous avons donc le début de la commande à utiliser pour déchiffrer notre fichier, déchiffrons-le !

```
mizaru@DESKTOP-23T8VRF:/mnt/c/Users/sMizaru/Downloads$ openssl enc -aes-128-ctr -d -K 00112233445566778899aabbccddeeff -iv 00000000000000000000000000000000 -in flag.jpg.enc -out flag.jpg  
```

<br>

Ouvrons maintenant le fichier flag.jpg et ?

<img src="https://img.onii.wtf/i/hh1su.png">

*Surprise! Nous avons le flag !*

<br>

<br>

Un challenge assez simple mais qui méritait d'être flag, assez intéressant.
