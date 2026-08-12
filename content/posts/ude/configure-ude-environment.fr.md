---
title: "Créer et configurer l'Unified Development Environment"
date: 2026-08-12
draft: false
categories: ["Admin & Power Platform"]
tags: ["UDE", "Power Platform", "Centre d'administration Power Platform", "Dataverse", "Environnements"]
summary: "Microsoft remplace progressivement les Cloud Hosted Environments (CHE) gérés via LCS par UDE, un environnement de développement connecté nativement à Power Platform. Voici comment le configurer."
---

## Contexte
Pendant des années, développer sur Dynamics 365 F&O signifiait provisionner un **Cloud Hosted Environment (CHE)** à partir de Lifecycle Services (LCS) : une machine virtuelle dédiée qui devait être corrigée, maintenue, et qui continuait souvent à facturer les frais de stockage même après l'arrêt.

Microsoft a introduit une nouvelle approche : l'**Unified Developer Experience (UDE)**. L'idée centrale est d'exécuter les applications finance et opérations en tant qu'application hébergée par **Microsoft Dataverse**, aux côtés de Power Apps ou Power Automate — plutôt que en tant que système ERP isolé nécessitant sa propre infrastructure.

Concrètement, avec UDE :
- L'édition et le débogage du code se font toujours localement, dans Visual Studio.
- Mais l'**exécution du code se fait dans le cloud** — plus de VHD à gérer, plus de VM à maintenir en cours d'exécution.
- L'approvisionnement et l'administration se font via le **Centre d'administration Power Platform (PPAC)** au lieu de LCS.
- Le déploiement suit désormais une approche basée sur les pipelines (Azure DevOps), intégrée à l'écosystème Power Platform plus large

| | CHE (modèle précédent) | UDE (nouveau modèle) |
|---|---|---|
| Approvisionnement | LCS | Centre d'administration Power Platform |
| Infrastructure | Machine virtuelle dédiée à gérer | Gérée par Microsoft |
| Exécution du code | Sur la machine virtuelle | Dans le cloud |
| Coût au repos | Le stockage peut continuer à facturer | Optimisé, pas de VHD à stocker |
| Intégration Power Platform | Faible / manuelle | Native (Dataverse, Power Apps, Power Automate) |

## Prérequis
- En tant que prérequis, vous devez avoir accès à un environnement sandbox provisionné et orienté vers le développement.
- Le compte utilisateur que vous utiliserez pour le développement dans l'environnement sandbox doit se voir attribuer le rôle Administrateur système.
- La machine de développement doit disposer d'au moins 16 Go d'espace libre sur le lecteur système local pour télécharger l'extension et les métadonnées.
- La machine de développement exécutant Microsoft Windows 10 ou 11 doit avoir Visual Studio 2022 installé avec au moins la charge de travail de développement de bureau .NET, le SDK de modélisation et quelques autres composants.
- Ce SDK et d'autres composants peuvent être sélectionnés et installés à partir du volet des composants individuels du programme d'installation de Visual Studio. Reportez-vous aux composants Visual Studio requis.
- Microsoft SQL Server Express LocalDB est installé par défaut avec Visual Studio 2022, et vous devez valider que vous pouvez vous y connecter avec l'authentification Windows.

## Limitations connues
- Le nom de l'environnement ne peut pas dépasser 20 caractères. Cette limitation s'applique au runtime finance et opérations.
- Lors de l'installation de l'**application de provisioning Dynamics 365 Finance and Operations** via le centre d'administration Power Platform sur une organisation existante, vous pouvez rencontrer une erreur si l'organisation se trouve dans une région Azure non prise en charge au sein d'une région.
*L'erreur indique : "La région sélectionnée ne supporte pas le déploiement de l'application FnO."* Pour éviter cette erreur, vous pouvez demander à Microsoft de déplacer l'environnement vers une région prise en charge via un ticket de support, ou provisionner un nouvel environnement dans une autre région prise en charge.

## Étapes
### 1. Provisionner ou sélectionner un environnement dans le PPAC
1. Connectez-vous au [Centre d'administration Power Platform](https://admin.powerplatform.microsoft.com).
2. Allez dans **Environnements** et cliquez sur le bouton nouveau.
{{< img src="images/ude/image1.png" alt="Créer un nouvel environnement" >}}
Assurez-vous d'entrer un Nom unique, de choisir le type **Sandbox** et d'activer **Ajouter un magasin de données Dataverse**
3. Activez l'application Dynamic 365 sur l'écran suivant.
{{< img src="images/ude/image2.png" alt="Activer l'application Dynamic 365 sur l'écran suivant" >}}
4. Une fois l'environnement prêt, ouvrez son écran de détails et sélectionnez les applications Dynamics 365 dans les ressources en haut et installez.**Outils de plateforme Dynamics 365 Finance and Operations**
5. Une fois cela fait, nous devons installer une autre application : **l'application de provisioning Finance and Operations.**
Sélectionnez-la, cliquez sur suivant et vous serez dirigé vers une nouvelle page :
{{< img src="images/ude/image3.png" alt="Activer l'application Dynamic 365 sur l'écran suivant" >}}
Sélectionnez les cases à cocher **Activer les outils de développement pour Finance and Operations** et **Activer les données de démonstration pour Finance and Operations**, et sélectionnez la version appropriée du produit et cliquez sur Installer.
{{< img src="images/ude/image4.png" alt="Activer l'application Dynamic 365 sur l'écran suivant" >}}
Après l'installation, si vous voyez cette carte dans l'environnement PP, cela confirme que l'environnement est UDE et vous pouvez voir l'URL F&O.
{{< img src="images/ude/image5.png" alt="Activer l'application Dynamic 365 sur l'écran suivant" >}}
{{< img src="images/ude/image6.png" alt="Activer l'application Dynamic 365 sur l'écran suivant" >}}
5. Allez dans Visual Studio, sélectionnez Extensions > Gérer les extensions.
6. Installez l'extension Power Platform pour Visual Studio.
{{< img src="images/ude/image7.png" alt="Activer l'application Dynamic 365 sur l'écran suivant" >}}
7. Ouvrez à nouveau Visual Studio, vous devriez voir de nouveaux éléments sous le menu Outils.
{{< img src="images/ude/image8.png" alt="Activer l'application Dynamic 365 sur l'écran suivant" >}}
8. Cliquez sur Se connecter à Dataverse, Cochez Se connecter en tant qu'utilisateur actuel et Afficher la liste des organisations disponibles. Dans la liste affichée, choisissez votre environnement.
{{< img src="images/ude/image9.png" alt="Activer l'application Dynamic 365 sur l'écran suivant" >}}
9. Choisissez l'environnement UDE qui a été précédemment créé et cliquez sur connexion.
{{< img src="images/ude/image10.png" alt="Activer l'application Dynamic 365 sur l'écran suivant" >}}
10. En attente de la connexion de Visual Studio à l'environnement Dataverse et sélection d'une solution **(ce n'est pas important, choisissez le par défaut)**.
{{< img src="images/ude/image11.png" alt="Activer l'application Dynamic 365 sur l'écran suivant" >}}
Une fois connecté, il vous demandera si vous souhaitez télécharger les métadonnées, l'extension F&O pour VS et d'autres actifs, cliquez sur Oui.
{{< img src="images/ude/image12.png" alt="Activer l'application Dynamic 365 sur l'écran suivant" >}}
Le processus est maintenant largement automatisé, et vous pouvez suivre la progression dans la fenêtre de sortie.
{{< img src="images/ude/image13.png" alt="Activer l'application Dynamic 365 sur l'écran suivant" >}}

## Lectures complémentaires
- Documentation Microsoft : <a href="https://learn.microsoft.com/en-us/power-platform/developer/unified-experience/finance-operations-install-config-tools" target="_blank">Installer et configurer les outils de développement</a> 
- Prochain article : "Comment déployer votre code sur l'environnement UDE"
