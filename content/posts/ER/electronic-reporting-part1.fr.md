---
title: "Electronic Reporting dans D365 F&O - Partie 1 : les fondations et le modèle de données"
date: 2026-08-17
draft: false
categories: ["Configuration fonctionnelle"]
tags: ["Electronic Reporting", "ER", "Configuration"]
summary: "Electronic Reporting (ER) permet de générer ou d'importer des documents réglementaires sans écrire de code. Premier volet d'une série : les concepts clés, la mise en place des fournisseurs de configuration, et la création d'un modèle de données pas à pas."
series: "Electronic Reporting"
---
 
## Contexte
**Electronic Reporting (ER)** est l'outil de D365 F&O qui permet de générer des documents électroniques (factures, relevés bancaires, déclarations fiscales, exports réglementaires...) ou d'en importer, **sans écrire une ligne de code X++** dans la majorité des cas. C'est l'outil utilisé en interne par Microsoft pour livrer la plupart des formats de paiement et de conformité locale (SEPA, formats bancaires, déclarations fiscales par pays...).
 
Ce sujet est assez vaste pour mériter plusieurs articles. Voici le plan de la série :
- **Partie 1** : concepts clés, mise en place des fournisseurs de configuration, création d'un modèle de données.
- **Partie 2** : le mapping du modèle (relier le modèle de données aux tables réelles de D365).
- **Partie 3** : le designer de format, le mapping du format, et la génération du document final.

## Les concepts clés à comprendre avant de commencer 
ER repose sur une chaîne de composants, chacun avec un rôle précis :
 
| Composant | Rôle |
|---|---|
| **Fournisseur de configuration** (Configuration provider) | Identifie qui possède/maintient une configuration ER (Microsoft, un pays, ou ta propre entreprise) |
| **Modèle de données** (Data model) | Représentation abstraite, en termes métier, de ce que le document doit contenir — indépendante de la structure des tables D365 |
| **Mapping du modèle** (Model mapping) | Fait le lien entre le modèle de données abstrait et les vraies tables/champs/requêtes de D365 |
| **Format** | Décrit la structure du fichier de sortie (formats supporté TEXT, XML, JSON, PDF, Microsoft Word, Microsoft Excel, and OPENXML) |
| **Mapping du format** (Format mapping) | Relie chaque élément du format au modèle de données |
 
**Pourquoi cette séparation en couches ?** Parce qu'un même modèle de données (par exemple "Facture client") peut être réutilisé par plusieurs formats différents (un export XML pour un pays, un CSV pour un autre), et qu'un mapping de modèle peut être maintenu indépendamment des évolutions de version de D365 — c'est ce qui permet à Microsoft de livrer des mises à jour de configuration sans casser tes personnalisations.
 
## Prérequis
- Dynamics 365 F&O version 10.0.x.
- Le rôle de sécurité **Electronic reporting developer** (ou un rôle administrateur incluant ces droits).
- Accès à **Administration d'organisation > Espaces de travail > Electronic reporting**.

## Étape 1 : créer et activer un fournisseur de configuration
Avant de créer quoi que ce soit, il faut un fournisseur de configuration actif — c'est lui qui "possède" tes futures configurations.

1. Va dans **Administration d'organisation > Espaces de travail > Gestion des états électronic**.
2. Sélectionne **Fournisseur de configuration** dans la parties liens connexes.
{{< img src="images/ER/part1/image1.png" alt="config providers" >}}
3. Clique sur **New**.
4. Renseigne un **Nom** (ex : le nom de votre entreprise) et l'**URL** de votre entreprise.
{{< img src="images/ER/part1/image2.png" alt="config providers parameters" >}}
5. Dans l'espace de travail vous avez deux tuile de configuration.
6. Clique sur les 3 points de la tuile de votre entreprise et clique sur définir comme actif.
{{< img src="images/ER/part1/image3.png" alt="config providers set active" >}}

## Étape 2 : créer un modèle de données
Prenons un exemple concret et volontairement simple pour cette première partie : un modèle de données pour exporter les commandes de ventes.
 
1. Dans l'espace de travail **Electronic reporting**, sélectionne la tuile **Configuration des états**.
2. Clique sur **Créer la configuration**.
3. Choisis **Racine** (pour créer un nouveau modèle depuis zéro, sans repartir d'un modèle existant).
4. Renseigne :
   - **Nom** : `Sales Order Model`.
   - **Description** : `Modèle de données pour l'export de la liste des commandes de ventes`.
5. Le champ **Fournisseur de configuration** se remplit automatiquement avec celui que tu as activé à l'étape 1.
6. Clique sur **Créer la configuration**.
{{< img src="images/ER/part1/image4.png" alt="config providers set active" >}}

### Concevoir la structure du modèle
1. Sélectionne la configuration `Sales Order Model` fraîchement créée.
2. Clique sur le boutton **Concepteur**.
{{< img src="images/ER/part1/image5.png" alt="select model designer" >}}
3. Dans le designer, clique sur **Nouveau** pour ajouter un nœud racine :
   - **Name** : `SalesOrderModel`.
{{< img src="images/ER/part1/image6.png" alt="add root" >}}
4. Sélectionne ce nœud racine, puis clique de nouveau sur **New** pour ajouter un premier nœud enfant `SalesOrder` de type `Record List` :
Ce nœud affiche la liste des commandes de vente.
{{< img src="images/ER/part1/image9.png" alt="add root" >}}
5. Sélectionne le nœud `SalesOrder`, puis clique de nouveau sur **New** pour ajouter un premier champ enfant :
   - **Name** : `SalesId`.
   - **Type** : Chaine.
{{< img src="images/ER/part1/image7.png" alt="add root" >}}
6. Répète l'opération pour ajouter :
   - **Name** : `CustAccount`.
   - **Type** : Chaine.
7. Clique sur **Enregistrer**.
8. Ferme le designer, puis clique sur **Modifier le statut** pour passer la configuration de **Brouillon** à **Terminé(e)**.
{{< img src="images/ER/part1/image8.png" alt="chage status" >}}
**Pourquoi passer par "Change status" ?** Une configuration en statut "Draft" (brouillon) est en cours d'édition. Il faut la faire passer par au moins "Terminé(e)" pour qu'elle devienne sélectionnable comme source dans les étapes suivantes (mapping et format).
 
## Pièges courants
- **Oublier d'activer le fournisseur de configuration** avant de créer une configuration - c'est l'erreur numéro un pour quelqu'un qui découvre ER.
- **Confondre le modèle de données avec le format** : le modèle décrit le contenu en termes métier abstraits (ex: "Sales Order Model"), le format décrit la structure physique du fichier (ex: à quelle position, avec quel séparateur). Les deux sont volontairement séparés.
- **Laisser une configuration en statut "Draft"** : elle restera invisible dans les listes de sélection des autres composants tant qu'elle n'est pas passée à un statut plus avancé via "Modifier le statut".
- **Créer un nouveau modèle à chaque besoin** au lieu de vérifier si Microsoft en fournit déjà un proche pour votre scénario (paiements, factures...) - dans un projet réel, mieux vaut chercher une configuration existante à étendre plutôt que repartir de zéro.

## Pour aller plus loin
- [Vue d'ensemble d'Electronic Reporting (ER) — Microsoft Learn](https://learn.microsoft.com/dynamics365/fin-ops-core/dev-itpro/analytics/general-electronic-reporting)
- [Créer des configurations Electronic Reporting — Microsoft Learn](https://learn.microsoft.com/dynamics365/fin-ops-core/dev-itpro/analytics/electronic-reporting-configuration)
- **Partie 2 de cette série** : le mapping du modèle, pour relier `Sales Order Model` aux vraies données de `SalesTable`