---
title: "Electronic Reporting dans D365 F&O - Partie 3 : Concepteur de format et génération du rapport"
date: 2026-08-28
draft: true
categories: ["Configuration fonctionnelle"]
tags: ["Electronic Reporting", "ER", "Configuration"]
summary: "Le modèle et la correspondance sont prêts - il est maintenant temps de produire réellement un fichier. Voici comment concevoir un format Excel pour le modèle de commande client, le lier au modèle et générer un export réel."
---

## Contexte

Dans [Partie 1](/posts/electronic-reporting-partie-1-fondations-modele-donnees/), nous avons construit le `Modèle de commande client`, une structure abstraite avec `SalesId` et `CustAccount`. Dans [Partie 2](/posts/electronic-reporting-partie-2-model-mapping/), nous avons connecté cette structure aux données réelles en la mappant à `SalesTable`.

Aucune de ces deux pièces ne produit en elle-même un fichier réel. C'est le rôle du **format** : il décrit la forme physique de la sortie (un document XML, un fichier CSV, un classeur Excel...), et il est lié champ par champ au modèle de données — non pas directement au tableau. Cette dernière couche est celle qui transforme finalement les « données abstraites » en « un fichier que vous pouvez ouvrir ».

Dans cette partie, nous allons construire un format Excel simple pour le `Modèle de commande client` et générer un véritable export.

## Prérequis

- La configuration du `Modèle de commande client` de la Partie 1, statut **Terminée**.
- La configuration du `Mappage du modèle de commande client` de la Partie 2, statut **Terminée**.
- L'accès à **Administration organisationnelle > Espaces de travail > Electronic Reporting**.

## Étape 1 : Créer un modèle de fichier

1. Ouvrez un nouveau fichier Excel et créez un tableau contenant deux colonnes *Numéro de commande* et *Compte client*
{{< img src="images/ER/part3/image1.png" >}}

2. Définissez le *nom* du tableau :
    - Dans l'onglet **Formules** d'Excel, cliquez sur **Gestionnaire de noms** dans la section *Noms définis*.
    - Sélectionnez la ligne de tableau et cliquez sur le bouton **Modifier**.
    {{< img src="images/ER/part3/image2.png" >}}
    - Entrez un nom cohérent *SalesOrders* et cliquez sur le bouton **OK**
    {{< img src="images/ER/part3/image3.png" >}}

3. Définissez le nom de la colonne de tableau *Numéro de commande* et de toutes les colonnes de feuille si nécessaire.
{{< img src="images/ER/part3/image4.png" >}}

4. Répétez l'opération pour la colonne de tableau *Compte client*.

5. Enregistrez le modèle Excel.

## Étape 2 : Créer une configuration de format

1. Dans **Electronic Reporting**, sélectionnez la configuration `Modèle de commande client`.
2. Sélectionnez **Créer une configuration**.
3. Entrez un nom.
4. Sélectionnez le type de format **Excel**.
5. Sélectionnez la dernière **version du modèle de données** et sélectionnez la **définition du modèle de données**.
{{< img src="images/ER/part3/image5.png" >}}

6. Cliquez sur le bouton **Créer une configuration**.
{{< img src="images/ER/part3/image6.png" >}}

## Étape 3 : Joindre le modèle au format

Joignez le modèle Excel créé à l'étape 1 au format *Sales order (Excel)*.

1. Sélectionnez le format *Sales order (Excel)* et cliquez sur le bouton **Pièces jointes**.
{{< img src="images/ER/part3/image7.png" >}}

2. Dans l'écran **Pièces jointes**, cliquez sur le bouton **Nouveau** et cliquez sur **Fichier**.
{{< img src="images/ER/part3/image8.png" >}}

3. Téléchargez le fichier de modèle.

## Étape 4 : Créer des plages de format et des cellules

1. Ouvrez l'écran **Electronic Reporting**, sélectionnez le format *Sales order (Excel)*.
2. Cliquez sur le bouton **Concepteur**.
3. Dans l'écran *Concepteur de format*, ouvrez la liste déroulante **Ajouter une racine** et sélectionnez **Fichier** dans la catégorie *Excel*.
{{< img src="images/ER/part3/image9.png" >}}

4. Entrez un **nom**, sélectionnez le fichier de modèle joint et cliquez sur **OK**
{{< img src="images/ER/part3/image10.png" >}}

5. Sélectionnez le nœud racine et ouvrez la liste déroulante **Ajouter**.
6. Sélectionnez Plage (cela correspond aux lignes du tableau du modèle).
{{< img src="images/ER/part3/image11.png" >}}

7. Entrez un **nom** et dans le champ **Plage Excel**, entrez le même nom que celui défini dans le modèle.
8. Sélectionnez **Vertical** dans le champ direction de réplication pour afficher toutes les lignes de commande client, sinon il n'affichera que la première ligne.
{{< img src="images/ER/part3/image12.png" >}}
{{< img src="images/ER/part3/image13.png" >}}

9. Sélectionnez le nœud de plage, ouvrez la liste déroulante **Ajouter** et sélectionnez Cellule.
{{< img src="images/ER/part3/image14.png" >}}

10. Entrez un **nom** pour la première cellule Excel.
11. Dans le champ **Plage Excel**, entrez le même nom que celui défini dans le modèle et sélectionnez le type de données (Chaîne dans notre exemple).
{{< img src="images/ER/part3/image15.png" >}}

11. Répétez l'opération pour le compte client.

## Étape 5 : Mapper le format avec le modèle

1. Dans l'écran *Concepteur de format*, cliquez sur l'onglet **Mappage** à droite.
{{< img src="images/ER/part3/image16.png" >}}

2. Sélectionnez le nœud de format *SalesOrders* et le nœud de modèle *SalesOrder* et cliquez sur le bouton **Lier**.
{{< img src="images/ER/part3/image17.png" >}}

3. Répétez l'opération pour les sous-nœuds.
4. Cliquez sur **Enregistrer**.
5. Retournez à **Electronic Reporting**, sélectionnez le format *Sales order (Excel)* et puis sélectionnez **Modifier le statut** pour passer la configuration de **Brouillon** à **Terminée**.

## Étape 6 : Valider et exécuter le rapport

1. Sélectionnez le format *Sales order (Excel)* et cliquez sur le bouton **Valider**.
2. Cliquez sur le bouton **Exécuter** et cliquez sur **OK**
3. Le rapport affiche toutes les lignes de commande client dans le fichier Excel.
{{< img src="images/ER/part3/image18.png" >}}

Ceci est un exemple simple de la façon de créer un rapport Electronic Reporting. Nous pouvons également ajouter des paramètres utilisateur, afficher un volet de requête, ajouter une recherche multisélection, exécuter le rapport dans l'ensemble de l'entreprise et nous pouvons créer un menu pour exécuter le rapport en dehors de l'espace de travail Electronic Reporting.

## Lectures complémentaires

- [Vue d'ensemble de Electronic Reporting (ER) — Microsoft Learn](https://learn.microsoft.com/dynamics365/fin-ops-core/dev-itpro/analytics/general-electronic-reporting)
- [Créer des configurations Electronic Reporting — Microsoft Learn](https://learn.microsoft.com/dynamics365/fin-ops-core/dev-itpro/analytics/electronic-reporting-configuration)
