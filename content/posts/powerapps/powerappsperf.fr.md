---
title: "Performance de Power Apps : techniques pratiques pour corriger les applications canevas lentes"
date: 2026-08-31
draft: true
categories: ["Power Platform"]
tags: ["Power Apps", "Performance", "Applications canevas", "Power Fx", "Dataverse"]
summary: "Démarrage lent, écrans qui manquent de fluidité, actions retardées : les problèmes de performance des applications canevas proviennent souvent de quelques schémas récurrents. Voici comment les diagnostiquer et appliquer les corrections qui font réellement la différence."
---
 
## Contexte
 
Les applications canevas sont faciles à créer rapidement, mais cette même flexibilité facilite l'accumulation d'une dégradation de performance.
Cet article présente les causes récurrentes des applications canevas lentes, classées approximativement par ordre d'impact, ainsi que la correction concrète à appliquer dans chaque cas.
 
## Prérequis
 
- Une application canevas existante que vous souhaitez optimiser (ou que vous êtes en train de créer)
- Un accès à Power Apps Studio avec des autorisations de créateur.

## Étape 1 : charger les jeux de données simultanément
Les données sont chargées en se connectant au connecteur de la source de données. Le chargement séquentiel, par exemple à l'aide de plusieurs fonctions `ClearCollect` dans la propriété `OnStart`, ralentit considérablement l'application.
Pour réduire les temps de chargement, utilisez la fonction **Concurrent** : elle permet de charger les données (comme plusieurs collections) simultanément plutôt que séquentiellement.
{{< img src="images/powerapps/image1.png" >}}
 
## Étape 2 : mettre les données des tables en cache
Utilisez les fonctions **ClearCollect** et **Set** pour mettre localement en cache les données des tables et éviter de les charger dans les galeries, les formulaires, les contrôles de liste déroulante, etc., chaque fois qu'un écran est ouvert.
{{< img src="images/powerapps/image2.png" >}}
{{< img src="images/powerapps/image3.png" >}}

## Étape 3 : limiter la taille d'une collection
Limitez le nombre de colonnes d'une collection et sélectionnez uniquement celles que vous utilisez dans l'application.
{{< img src="images/powerapps/image4.png" >}}

## Étape 4 : effectuer les mises à jour en bloc
{{< img src="images/powerapps/image5.png" >}}
 
## Étape 5 : réduire les opérations effectuées au démarrage
Tout ce qui se trouve dans `App.OnStart` est exécuté avant que l'utilisateur ne voie quoi que ce soit. Plus cette formule est légère, plus le démarrage semble rapide.
 
- Remplacez les constantes stockées dans des variables définies dans `OnStart` par des **formules nommées**. Elles sont calculées à la demande, uniquement lorsqu'elles sont référencées, au lieu de l'être systématiquement à chaque lancement.
 
## Pièges courants
 
- **Optimiser les contrôles avant de vérifier les schémas de données** : un écran comportant 15 contrôles et une seule mauvaise boucle `ForAll` est généralement plus lent qu'un écran avec 60 contrôles correctement conçus. Commencez par établir un profil de performance.
- **Ne pas tenir compte des avertissements de délégation** : une application qui « fonctionne correctement » lors des tests avec 50 lignes peut retourner silencieusement des résultats incorrects en production avec 5 000 lignes, simplement parce qu'un filtre ne pouvait pas être délégué.

## Pour aller plus loin
- [Conseils et bonnes pratiques pour améliorer les performances des applications canevas — Microsoft Learn](https://learn.microsoft.com/power-apps/maker/canvas-apps/performance-tips)
- [Identifier et atténuer les problèmes de performance des applications canevas — Microsoft Learn](https://learn.microsoft.com/power-platform/architecture/key-concepts/performance/top-issues)
- [Optimiser les applications canevas qui nécessitent une logique métier complexe — Microsoft Learn](https://learn.microsoft.com/power-platform/architecture/reference-architectures/optimize-performance-canvas-apps)
