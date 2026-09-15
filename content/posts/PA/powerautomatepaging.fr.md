---
title: "Pagination dans power automate : récupérer plus de 5000 lignes"
date: 2026-09-15
draft: false
categories: ["Power Platform"]
tags: ["Power Automate", "Dataverse", "FetchXML", "Pagination"]
summary: "L'action List rows du connecteur Dataverse plafonne à 5000 lignes par appel. Voici comment construire une boucle de pagination robuste avec FetchXML, le paging cookie et le flag More Records pour récupérer un jeu de données complet, quelle que soit sa taille."
---
 
## Contexte
 
Dataverse impose une limite technique : **la requête `List rows` ne peut jamais retourner plus de 5000 lignes**, quelle que soit la façon dont tu construis ta requête (Filter rows ou FetchXML). Pour un jeu de données plus petit, ça ne se voit jamais. Pour une synchronisation, un export, ou un traitement en masse sur une table qui dépasse ce seuil, le flow s'arrête silencieusement à 5000 lignes sans erreur.
 
La solution : une boucle qui redemande la page suivante tant qu'il en reste, en s'appuyant sur deux informations que Dataverse renvoie avec chaque réponse :
 
- **`@Microsoft.Dynamics.CRM.morerecords`** — un booléen qui indique s'il reste des lignes à récupérer.
- **`@Microsoft.Dynamics.CRM.fetchxmlpagingcookie`** — un curseur de pagination, plus performant qu'un simple numéro de page, à réinjecter dans la requête suivante.

## Prérequis
 
- Un flow Power Automate avec une action **List rows** (connecteur Dataverse) configurée en mode **Fetch XML Query** ou **Filter rows**

## Étape 1 : initialiser les variables de pagination.
 
Avant la boucle, ajoute quatre variables :
 
| Nom | Type | Valeur initiale |
|---|---|---|
| `PageNumber` | Integer | `1` |
| `PagingCookie` | String | *(vide)* |
| `MoreRecords` | Boolean | `true` |
| `TotalRecordsCount` | Integer | `0` |

{{< img src="images/PA/image1.png">}}

## Étape 2 : Construire le squelette de la boucle
 
Ajoute une action **Do until**, avec comme condition : `MoreRecords` est égal à `false`.
{{< img src="images/PA/image2.png">}}

**Point d'attention important avant d'aller plus loin :** le nombre d'itérations par défaut d'un `Do until` est plafonné (60 par défaut). Si ta table peut dépasser 300 000 lignes, va dans les paramètres de l'action (l'icône ⚙️) et augmente la limite du nombre d'itérations — sinon la boucle s'arrêtera prématurément sans avoir tout récupéré, sans erreur explicite non plus.
 
## Étape 3 : La requête FetchXML dynamique
 
À l'intérieur de la boucle, place ton action **List rows**, avec une requête FetchXML qui inclut les attributs `page`,  `paging-cookie` dynamiquement et `count` définie à 5000 (la limite de Dataverse):
 
```xml
<fetch count="5000" page="@{variables('PageNumber')}" @{if(equals(variables('PageNumber'), 1), '', if(equals(variables('PagingCookie'), ''), '', concat('paging-cookie=''', encodeXmlValue(variables('PagingCookie')), '''')))}>
  <entity name="kpmg_movies">
    <attribute name="kpmg_categories" />
    <attribute name="kpmg_name" />
  </entity>
</fetch>
```
Remarque la syntaxe : `variables('PagingCookie')` s'insère **directement dans la balise `<fetch>`** si la variable est vide, donc rien n'est injecté. Sur les suivantes, elle contient littéralement `paging-cookie="..."`.
 
## Etape 4 : Ajoute une condition pour vérifier s'il y a des données.
Toujours dans la boucle, après l'action `List rows` :
- Ajoute une condition qui vérifie si l'action `List rows` contient des données avec cette expression: 
Si **`length(outputs('List_movies')?['body/value'])`** est supérieur à 0.
{{< img src="images/PA/image3.png">}}

## Étape 5 : mettre à jour les variables de contrôle
**Si la condition est à True:**

- Mettre à jour la variable **`PagingCookie`**.
C'est l'étape la plus délicate de tout le processus : la valeur brute renvoyée par Dataverse dans `@Microsoft.Dynamics.CRM.fetchxmlpagingcookie` n'est pas directement réutilisable telle quelle — elle est encodée et contient des caractères qu'il faut décoder avant de la réinjecter dans le XML de la requête suivante:
1. Vérifie que la propriété **`@Microsoft.Dynamics.CRM.fetchxmlpagingcookie`** n'est pas nulle.
2. Extrait uniquement la partie utile de la chaîne brute.
3. Décode les caractères spéciaux (`<`, `>`, `"`) qui ont été échappés au format HTML:

**`decodeUriComponent(decodeUriComponent(split(split(outputs('List_movies')?['body']?['@Microsoft.Dynamics.CRM.fetchxmlpagingcookie'], 'pagingcookie="')[1], '" istracking')[0]))`**

- Incrémente la variable **`PageNumber`** de 1.
- Mettre à jour la variable **`MoreRecords`** à *true*.
- Incrémente la variable **`RecordsCount`** de la longueur du tableau de résultats (`length(outputs('List_rows')?['body/value'])`).

**Si la condition est à FALSE:**
- Mettre à jour la variable **`MoreRecords`** à *false* ce qui permet d'arrêter la boucle `Do until`.

{{< img src="images/PA/image4.png">}}

## Étape 6 : traiter les résultats de chaque page
 
N'attends pas la fin de la boucle pour traiter les données — ajoute ton traitement (mise à jour d'une autre table, envoi vers une API, écriture dans un fichier...) **à l'intérieur de la boucle**, juste après `List rows`, page par page.
 
## Pièges courants
- **Laisser le réglage natif "Pagination" activé dans les paramètres de `List rows`** en même temps qu'on gère la pagination manuellement via `page`/`paging-cookie` : les deux mécanismes entrent en conflit — désactive l'option native de pagination si tu gères tout toi-même via FetchXML.
- **Utiliser un tri (`order`) sur un attribut d'une entité liée (`link-entity`)** : certaines requêtes triées de cette façon ne supportent pas le paging cookie, et Dataverse retombe silencieusement sur une pagination "simple" par numéro de page uniquement — moins performante, et à surveiller si tes résultats semblent incohérents entre deux pages.
- **Oublier d'augmenter la limite d'itérations du `Do until`** pour un très gros volume : la valeur par défaut est pensée pour des cas simples, pas pour des exports de plusieurs centaines de milliers de lignes.
- **Traiter un `MoreRecords` absent comme `true` par défaut** : si l'expression qui lit ce champ ne gère pas le cas où il est vide/absent, la boucle peut ne jamais se terminer.

## Pour aller plus loin
- [Page large result sets (Dataverse) in Power Automate — Power Platform Community](https://community.powerplatform.com/blogs/post/?postid=0864e145-da42-4922-b667-ff10dd160089)