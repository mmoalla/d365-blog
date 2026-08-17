---
title: "Rapports électroniques dans D365 F&O - Partie 2 : mappage du modèle"
date: 2026-08-17
draft: true
categories: ["Configuration fonctionnelle"]
tags: ["Rapports électroniques", "ER", "Configuration"]
summary: "Le modèle de données que nous avons créé dans la partie 1 n'est qu'une structure abstraite sans vraies données derrière. Le mappage du modèle est ce qui le relie à une table D365 réelle - voici comment connecter le modèle SalesOrder à SalesTable, champ par champ."
---

## Contexte

Dans la [Partie 1 de cette série](/posts/electronic-reporting-part1/), nous avons créé le `Modèle de commande client` : un modèle de données avec un nœud racine `SalesOrderModel` et deux champs, `SalesId` et `CustAccount`. À ce stade, le modèle est purement structurel — il décrit *ce que* le document devrait contenir, mais il ne sait pas *d'où* ces données proviennent réellement dans D365.

Dans cet article, nous configurerons le mappage de la source de données du modèle et mapperons le modèle précédemment créé aux données correspondantes.

## Prérequis

La configuration du `Modèle de commande client` de la Partie 1, avec le statut **Terminée**
- Le même rôle de sécurité **Développeur de rapports électroniques** utilisé dans la Partie 1
- Accès à **Administration de l'organisation > Espaces de travail > Rapports électroniques**.

## Étape 1 : Créer une configuration de concepteur de mappage de modèle

1. Dans **Rapports électroniques**, sélectionnez la configuration `Modèle de commande client`.
2. Cliquez sur le bouton **Concepteur**
3. Dans l'écran du concepteur SalesOrderModel, cliquez sur **Mapper le modèle à la source de données**.
4. Dans l'écran *Mappage du modèle à la source de données*, cliquez sur le bouton **Nouveau** pour ajouter un mappage.
{{< img src="images/ER/part2/image1.png" >}}
5. Choisissez la **Définition** `SalesOrderModel` créée à la partie 1 et entrez un nom.
6. Cliquez sur **Enregistrer**.
7. Dans le même écran *Mappage du modèle à la source de données*, cliquez sur le bouton **Concepteur**.
Dans l'écran du *Concepteur de mappage de modèle* est divisé en 3 panneaux :
    - Le panneau de gauche affiche différents **Types de sources de données** : Enregistrements de table, Énumération, Classe, Champ calculé, Paramètre d'entrée utilisateur, etc.
    - Le panneau du milieu affiche les **Sources de données** utilisées pour afficher les données dans le rapport.
    - Le panneau de droite contient le **Modèle de données** que nous avons configuré à la partie 1.
8. Sélectionnez **Enregistrements de table** dans le panneau de gauche et cliquez sur **Ajouter racine**
{{< img src="images/ER/part2/image2.png" >}}
9. Sélectionnez la table **SalesTable** et entrez un nom *SalesTable*.
10. Cliquez sur **OK**.
{{< img src="images/ER/part2/image3.png" >}}

## Étape 2 : Mapper les sources de données au modèle de données

1. Sélectionnez **SalesTable** Enregistrements de table dans le panneau des sources de données.
2. Sélectionnez **Commande client** Liste d'enregistrements dans le panneau du modèle de données.
3. Cliquez sur **Lier**
{{< img src="images/ER/part2/image4.png" >}}
*La liste des enregistrements de commande client sera maintenant en gras. Cela signifie qu'elle est liée aux enregistrements de salesTable*.
4. Développez **SalesTable** Enregistrements de table, trouvez le champ de table *Compte client* et sélectionnez-le.
5. Développez **Commande client** Liste d'enregistrements et sélectionnez le nœud *CustAccount*.
6. Cliquez sur **Lier**.
{{< img src="images/ER/part2/image5.png" >}}
7. Répétez l'opération pour le nœud SalesId :
    - Commande client avec SalesId.
8. Cliquez sur **Enregistrer**.
9. Retournez à **Rapports électroniques**, sélectionnez la configuration `Modèle de commande client` et puis sélectionnez **Modifier le statut** pour déplacer la configuration de **Brouillon** à **Terminée**.

## Lectures supplémentaires

- [Vue d'ensemble des rapports électroniques (ER) — Microsoft Learn](https://learn.microsoft.com/dynamics365/fin-ops-core/dev-itpro/analytics/general-electronic-reporting).
- [Sources de données des rapports électroniques — Microsoft Learn](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/analytics/er-data-sources).
- **Partie 2 de cette série** : mappage du modèle, pour lier le `Modèle de commande client` aux données réelles dans `SalesTable`.
