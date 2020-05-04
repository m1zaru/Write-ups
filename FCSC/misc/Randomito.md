<center><h1>FCSC 2020 - Randomito Writeup</h1></center>
<br>
<br>
<img src="https://img.onii.wtf/i/f9kbj.png">
<br>

Alalalahhh, Randomito, nous avons déjà un fichier randomito.py, regardons un peu son contenu : 

```python

#!/usr/local/bin/python2

import sys
import signal
from random import randint

# Time allowed to answer (seconds)
DELAY = 10

def handler(signum, frame):
   raise Exception("Time is up!\n")

def p(s):
	sys.stdout.write(s)
	sys.stdout.flush()

def challenge():

	for _ in range(10):
		p("[+] Generating a 128-bit random secret (a, b)\n")
		secret_a = randint(0, 2**64 - 1)
		secret_b = randint(0, 2**64 - 1)
		secret   = "{:016x}{:016x}".format(secret_a, secret_b)
		p("[+] Done! Now, try go guess it!\n")
		p(">>> a = ")
		a = int(input())
		p(">>> b = ")
		b = int(input())
		check = "{:016x}{:016x}".format(a, b)
		p("[-] Trying {}\n".format(check))
		if check == secret:
			flag = open("flag.txt").read()
			p("[+] Well done! Here is the flag: {}\n".format(flag))
			break
		else:
			p("[!] Nope, it started by {}. Please try again.\n".format(secret[:5]))

if __name__ == "__main__":
	signal.alarm(DELAY)
	signal.signal(signal.SIGALRM, handler)
	try:
		challenge()
	except Exception, e: 
		exit(0)
	else:
		exit(0)

```

On voit déjà une génération aléatoire d'un secret 128 bits qu'il faut deviner, mais l'erreur ici a été d'utiliser input() à la place de raw_input(), donc nous avons juste à entrer les variables de génération pour avoir le flag.

```
mizaru@DESKTOP-23T8VRF:~/web$ nc challenges2.france-cybersecurity-challenge.fr 6001                                                                                     
[+] Generating a 128-bit random secret (a, b)                                                                                                                           
[+] Done! Now, try go guess it!                                                                                                                                         
>>> a = secret_a                                                                                                                                                        
>>> b = secret_b                                                                                                                                                        
[-] Trying 8142edefbbba10d62e8f07a68f1ba070                                                                                                                             
[+] Well done! Here is the flag: FCSC{4496d11d19db92ae53e0b9e9415d99d877ebeaeab99e9e875ac346c73e8aca77}  
```
