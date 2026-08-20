---
title: "Electronic Reporting dans D365 F&O - Partie 2 : mapping du modèle"
date: 2026-08-20
draft: false
categories: ["Configuration fonctionnelle"]
tags: ["Electronic Reporting", "ER", "Configuration"]
summary: "Le modèle de données qu'on a construit dans la Partie 1 n'est qu'une structure abstraite, sans données réelles derrière. Le mapping du modèle est ce qui le relie à une vraie table D365 - voici comment relier Sales Order Model à SalesTable, champ par champ."
---
 
## Contexte
 
Dans la [Partie 1 de cette série](/posts/electronic-reporting-part1/), on a construit `Sales Order Model` : un modèle de données avec un nœud racine `SalesOrderModel` et deux champs, `SalesId` et `CustAccount`. À ce stade, le modèle est purement structurel — il décrit *ce que* le document doit contenir, mais il ne sait pas encore *d'où* viennent réellement ces données dans D365.
 
Dans cet article, on va configurer le mapping du modèle vers une source de données et relier le modèle créé précédemment aux données correspondantes.
 
## Prérequis
 
- La configuration `Sales Order Model` de la Partie 1, au statut **Completed**
- Le même rôle de sécurité **Electronic reporting developer** utilisé dans la Partie 1
- Accès à **Organization administration > Workspaces > Electronic reporting**
## Étape 1 : créer une configuration de mapping du modèle
 
1. Dans **Electronic reporting**, sélectionne la configuration `Sales Order Model`
2. Clique sur le bouton **Designer**
3. Sur l'écran du designer de `SalesOrderModel`, clique sur **Map model to datasource**
4. Sur l'écran *Model to datasource mapping*, clique sur le bouton **New** pour ajouter un mapping
{{< img src="images/ER/part2/image1.png" >}}
5. Choisis la **Definition** `SalesOrderModel` créée dans la partie 1 et saisis un nom
6. Clique sur **Save**
7. Sur ce même écran *Model to datasource mapping*, clique sur le bouton **Designer**
L'écran *Model mapping designer* est divisé en 3 panneaux :
- Le panneau de gauche affiche les différents **types de source de données** : Table records, Enumeration, Class, Calculated field, User input parameter, etc.
- Le panneau du milieu affiche les **sources de données** utilisées pour alimenter le document
- Le panneau de droite contient le **modèle de données** configuré à la partie 1
8. Sélectionne **Table records** dans le panneau de gauche et clique sur **Add root**
{{< img src="images/ER/part2/image2.png" >}}
9. Sélectionne la table **SalesTable** et saisis un nom, *SalesTable*
10. Clique sur **OK**
{{< img src="images/ER/part2/image3.png" >}}
## Étape 2 : relier les sources de données au modèle
 
1. Sélectionne **SalesTable** Table records dans le panneau des sources de données
2. Sélectionne **Sales Order** Record List dans le panneau du modèle de données
3. Clique sur **Bind**
{{< img src="images/ER/part2/image4.png" >}}
*Le Sales Order record list apparaît maintenant en gras. Cela signifie qu'il est relié (bindé) à SalesTable Records.*
 
4. Déplie **SalesTable** Table records, trouve le champ de table *Customer account* et sélectionne-le
5. Déplie **Sales Order** Record List et sélectionne le nœud *CustAccount*
6. Clique sur **Bind**
{{< img src="images/ER/part2/image5.png" >}}
7. Répète l'opération pour le nœud SalesId :
   - Sales Order avec SalesId
8. Clique sur **Save**
9. Retourne dans **Electronic reporting**, sélectionne la configuration `Sales Order Model`, puis sélectionne **Change status** pour faire passer la configuration de **Draft** à **Completed**
## Pour aller plus loin
 
- [Vue d'ensemble d'Electronic Reporting (ER) — Microsoft Learn](https://learn.microsoft.com/dynamics365/fin-ops-core/dev-itpro/analytics/general-electronic-reporting)
- [Sources de données Electronic Reporting — Microsoft Learn](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/analytics/er-data-sources)
- **Partie 3 de cette série** : Concepteur de format et génération du rapport