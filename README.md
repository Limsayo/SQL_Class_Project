# SQL_Class_Project

# Enquête sur un meurtre commis le 15/01/2018

## Glossaire des commandes et mots clés SQL Utilisées : 

Forme générale SELECT x FROM y WHERE machin == SELECTIONNER x DEPUIS y OU machin (je sais je simplifie)  

- **SELECT** : extrait les données de la base de données et les renvoie a l'utilisateur (nous)
- **FROM** : permet de sélectionner la base de donnée concernée
- **WHERE**: condition de la recherche  
- **AND** : ajoute une condition
- **LIKE** : cherche une suite de charactères dans un tableau (ici par exemple nous avons cherché une personne a l'ai de son prenom dans une colonne qui contient nom + prénom associés)
- **ORDER BY** : organise les résultats (croissants ou décroissant par exemple)
- **DESC** : ordre qu'on peut donner a ORDER BY = descendant
- **%** : Stand in qui signifique qu'il peut y avoir d'autre caractères ou rien a sa place/après

## Analyse du rapport de police sur le meurtre

Pour commencer j'ai téléchargé la base de donnée puis l'ai ouverte dans Vscode. J'ai rapidement survolé la base de données pour me familiariser avec son contenu (le type d'informations, les tables etc).

On cherche donc un(ou plusieurs ?) meurtriers qui a agi le 15/01/2018, le format de date utilisé est le suivant 20180115.

On a a notre disposition : les rapports de scènes de crime, les informations de la salle de sport 'get fit now', la liste de personnes qui ont check in a des evenements facebook variés, des rapports d'interrogatoires, une liste de personnes et de permis de conduire.

La première piste a explorer est d'isoler le meurtre dans la table crime_scene_report' et pour ce faire j'utilise :

```sql
SELECT description FROM crime_scene_report WHERE type = 'murder' AND date = 20180115 AND city = 'SQLcity;
 ```

pour trouver l'identification faite du meurtrier que l'on cherche. J'ai passé beaucoup de temps a comprendre que cette requête ne marchait pas parce que 'SQLcity' n'est pas 'SQL city' qui n'est pas non plus SQL City' ... En corrigeant la requête on obtient un seul résultat : 

> **Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".**


## Recherche du premier témoin

La prochaine chose a faire sera donc de trouver les informations que les témoins peuvent fournir. Les informations disponibles dans 'person' sont : un id unique, l'id de leur permis, le numéro d'adresse, le nom de rue, et un numéro de sécurité sociale. On dispose d'un nom et de l'adresse associée ainsi que de l'adresse d'un autre témoin non identifié.

Dans la table interview, on a trois colonnes : l'id de la personne interrogée,son nom ainsi que la transcription de l'interrogatoire.

En sachant cela sur les deux tables on peut chercher notre premier témoin et son témoignage. Pour ce faire on cherche a trouver une 'Annabel' qui vit sur Franklin ave.

Donc :  
```sql 
SELECT id FROM person WHERE address_street_name =  'Franklin Ave' AND name LIKE 'Annabel'; 
``` 
qui nous rend une parse error. Je vérifie donc la syntaxe sur w3schools.com.

la version correcte était 
```sql 
SELECT id FROM person WHERE address_street_name =  'Franklin Ave' AND name LIKE 'Annabel%'; 
```  
On obtient le numéro : 

> **16371**

## Témoignage 1

Avec ce numéro on trouve le témoignage d'Annabel dans la table 'interview'. La requête utilisée est la suivante : 
```sql 
SELECT transcript FROM interview WHERE person_id = 16371; 
``` 
On découvre ainsi que :

> **I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.**

Notre suspect va donc a la même salle de sport qu'Annabel et y était le 09/01/2018.

## Témoin 2 essai 1

Pour ce qui est du second témoin je ne sais pas trop quoi en faire a ce moment de l'enquête. Je cherche toutes les personnes qui vivent à 'Northwestern Dr' avec : 
```sql
 SELECT id, name, FROM person WHERE adress_street_name = 'Northwestern Dr'; 
 ``` 
Je trouve beaucoup de noms, j'utilise donc la commande 'COUNT' pour déterminer le nombre decandidats potentiels.

 ```sql 
 SELECT COUNT (name) FROM person WHERE address_street_name = 'Northwestern Dr'; 
 ``` 

> J'obtiens le nombre 50.

Je n'ai pas d'idées pour affiner ma recherche, j'utilise donc uniquement le premier témoignage pour poursuivre mes recherches et reviendrai ici si je bloque par ailleurs.

## Recherche du suspect

Notre suspect va donc a la même salle de sport qu'Annabel et y était le 09/01/2018.
La salle de sport est décrite par deux tables : get_fit_now_member et get_fit_now_check_in. On regarde donc qui était présent le 09/01/2018 dans get_fit_now_check_in. 

```sql 
SELECT membership_id FROM get_fit_now_check_in WHERE check_in_date = 20180109; 
```
On obtien dix ID : 
 >|X0643 | UK1F2 | XTE42 | 1AE2H | 6LSTG | 7MWHJ | GE5Q8 | 48Z7A | 48Z55 | 90081 | 
 >|------|-------|-------|-------|-------|-------|-------|-------|-------|-------|

## Témoin 2 essai 2

Ca a été rapide mais on bloque a nouveau.Apres un effort de réflexion + relecture je réalise avoir raté une information. Relisons ensemble le résultat de ma première requête : Security footage shows that there were 2 witnesses. The first witness lives at the **_last_** house on "Northwestern Dr". J'adapte donc ma recherche de témoins sur Northwestern Dr. 

```sql 
SELECT id, name, address_number FROM person WHERE address_street_name = 'Northwestern Dr' ORDER BY address_number DESC ; 
``` 

>| Id    | Name           | Adress number |
>|-------|----------------|---------------|
>| 14887 | Morty Schapiro | 4919          |
>|


Notre témoin sera donc le premier de la liste : Morty Schapiro. De la même façon que pour Annabel on cherche son témoignage : 

```sql 
SELECT transcript FROM interview WHERE person_id = 14887;
```

## Témoignage 2

On obtient son témoignage : 

> **'I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".**'

#### Beaucoup d'informations ici:

- Le suspect est effectivement membre de get fit now
- Son numero de membre commence par 48Z, cela réduit la liste des suspects a deux car le témoignage d'Annabel nous a déja permi de réduire le pool de suspects.
- 48Z correspond a un abonnement 'gold'
- La plaque d'immatriculation de sa voiture commence par H42W

En analysant l'autre témoignage j'ai trouvé deux personnes dont le numéro de membre commense par 48Z, les membres 48Z55 et 48Z7A. Je vais donc chercher leurs id dans la table get_fit_now_member puis trouver leurs plaques d'immatriculation dans drivers_licence.

```sql 
SELECT person_id, name, id FROM get_fit_now_member WHERE id LIKE '48Z%'; 
```
Je trouve: 

|Joe Germuska 
gym id = 48Z7A id= 28819 et Jeremy Bowers gym id = 48Z55 id = 67318 avec ces résultats nous pouvons continuer.


Dans la table drivers_licence je cherche donc:
```sql 
SELECT plate_number, name WHERE id IN (28819, 67318) AND plate_number LIKE '%H42w%; 
```
 mais c'est aller trop vite en besogne, le id de cette table n'est pas person_id. on retourne donc dans person et on trouve nos suspects : 
 ```sql
  SELECT name, license_id FROM person WHERE id IN (28819, 67318); 
  ``` 

#### J'obtient:

- Joe Germuska license_id = 173289
- Jeremy Bowers license_id = 423327

on utilise la requête d'avant en remplaçant les id et on a une correspondence entre l'id 423327 et la plaque 0H42W2. Notre coupable serait donc Jeremy Bowers. Je vérifie dans intrview : 
```sql 
SELECT transcript FROM interview WHERE person_id = 67318; 
``` 
(j'utilise son person_id). Son témoignage : 

**I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.**

## Recherche du commanditaire

Ce meurtre était une commande, il s'agit donc maintenant de trouver le/la commanditaire. A l'aide de la table drivers_license on utilise sa description phyisique pour trouver son numéro de permis. 
```sql 
SELECT id FROM drivers_license WHERE gender = 'female'AND hair_color = 'red' AND car_make = 'Tesla' AND car_model = 'Model S'; 
```
Cela nous donne trois id : 
- **202298**
- **291182**
- **918773**

On utilise ces trois id pour trouver les noms/ids associés dans persons 

```sql 
SELECT id, name FROM person WHERE license_id IN (202298, 291182, 918773); 
``` 

on trouve :
- **Red Korb id = 78881**
- **Regina George id = 90700**
- **Miranda Priestly id = 99716** <- **ndlr** vu la référence a _le diable s'habille en prada_ j'imagine que ce sera notre commanditaire

Il ne reste plus qu'a vérifier laquelle de ces personnes est allées trois fois a la SQL symphony : 

```sql 
SELECT person_id, count(*) as nombre_de_visites FROM facebook_event_checkin WHERE person_id IN (78881, 90700, 99716) AND event_name = 'SQL Symphony Concert' AND date LIKE '201712%' GROUP BY person_id; 
```

Une seule personne est allée à cet évènement 3 fois et l'id associé est 99716, c'est a dire celui de Miranda Priestly

Je cherche son interrogatoire : 

```sql 
SELECT transcript FROM interview WHERE person_id =99716 
```
et ne trouve pas de résultat, il semblerait que la police n'en soit pas arrivée à ce stade de l'enquête.

# Recommendations :

Informer la police de ces découvertes. Arrêter Miranda Priestly
