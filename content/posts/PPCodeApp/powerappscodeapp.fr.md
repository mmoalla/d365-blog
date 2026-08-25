---
title: "Power Apps Code Apps : créer des applications React pro-code dans Power Platform"
date: 2026-08-25
draft: false
categories: ["Power Platform"]
tags: ["Power Apps", "Code Apps", "React", "TypeScript", "Dataverse"]
summary: "Les Code Apps vous permettent d'écrire une application web React/TypeScript complète dans votre IDE et de l'exécuter dans Power Platform, avec un accès typé à Dataverse et à plus de 1 500 connecteurs, sans canvas en glisser-déposer ni code passe-partout PCF. Voici comment créer, connecter et exécuter votre première application."
---
 
## Contexte
 
Les applications canvas et les applications pilotées par modèle sont créées dans Power Apps Studio. Les **Code Apps** suivent une approche totalement différente : vous écrivez une application **React** autonome dans un véritable IDE comme VS Code, et la CLI Power Platform l'intègre à la plateforme. L'authentification, les connexions aux données et l'hébergement sont pris en charge pour vous, tandis que vous gardez un contrôle total sur l'interface et la logique, dans du code réel.
 
Microsoft a annoncé la disponibilité générale des Code Apps en février 2026. Il est important de préciser ce qu'elles sont et ce qu'elles ne sont pas :
 
- Ce ne sont pas des applications canvas, pas des applications pilotées par modèle et pas des contrôles PCF.
- Ce sont des applications web autonomes, créées par défaut avec *Vite, React et TypeScript*, qui s'exécutent dans l'environnement d'exécution Power Apps.
- Elles offrent un accès typé à **Dataverse et à plus de 1 500 connecteurs** directement depuis *JavaScript/TypeScript*, sans que vous ayez à écrire vous-même le code d'authentification.

Si vous êtes développeur et avez toujours trouvé les applications canvas trop limitées, sans pour autant vouloir renoncer à la gouvernance et à l'écosystème de connecteurs de Power Platform, cette option a été conçue pour répondre à ce besoin.
 
## Prérequis
 
- [Visual Studio Code](https://code.visualstudio.com/download).
- [Node.js](https://nodejs.org/).
- La **CLI Power Platform** (`pac`).
- Un environnement Power Platform dans lequel les Code Apps sont activées.
- Une licence **Power Apps Premium**.

## Étape 1 : Activer la fonctionnalité Code Apps
1. Ouvrez le [Centre d'administration Power Platform](https://admin.powerplatform.microsoft.com/).
2. Sélectionnez l'environnement dans lequel vous allez créer votre Code App.
3. Cliquez sur **Paramètres**.
4. Dans le menu **Produit**, cliquez sur le sous-menu **Fonctionnalités**.
5. Faites défiler la page et activez la fonctionnalité **Power Apps Code Apps**.
{{< img src="images/PPCodeApp/image1.png" >}}

## Étape 2 : Créer le squelette du projet à partir du modèle officiel
 
```bash
npx degit github:microsoft/PowerAppsCodeApps/templates/vite my-code-app
cd my-code-app
```
 
Vous obtenez ainsi un projet Vite + React + TypeScript propre, avec le SDK Power Apps déjà intégré. Aucune configuration manuelle de l'infrastructure du SDK n'est nécessaire.
{{< img src="images/PPCodeApp/image2.png" >}}
 
## Étape 2 : S'authentifier et sélectionner l'environnement cible
 
```bash
pac auth create
pac env list
pac env select --environment <your-environment-id>
```
{{< img src="images/PPCodeApp/image3.png" >}}
 
## Étape 3 : Installer les dépendances et initialiser l'application
 
```bash
npm install
pac code init --displayname "My First Code App"
```
{{< img src="images/PPCodeApp/image4.png" >}}

`pac code init` enregistre l'application dans votre environnement Power Platform et génère un fichier `power.config.json`. C'est ce fichier qui relie votre projet local à la plateforme, aussi bien pour l'accès aux données que pour le déploiement ultérieur.
{{< img src="images/PPCodeApp/image5.png" >}}

## Étape 4 : Connecter une source de données

C'est ici que les Code Apps se distinguent vraiment d'une application React classique : au lieu d'écrire manuellement des appels API, vous générez depuis la CLI un accès typé à n'importe quel connecteur Power Platform.
```bash
pac code add-data-source -a "shared_office365users" -c "<connector_id>"
```
Cette commande génère les fichiers suivants :
{{< img src="images/PPCodeApp/image6.png" >}}
 
## Étape 5 : L'exécuter localement
 
Comme les Code Apps sont basées sur Vite, le développement local fonctionne exactement comme dans tout projet React moderne, avec notamment le rechargement à chaud :
 
```bash
npm run dev
```
L'application utilise la source de données réellement connectée pendant le développement. Ce que vous voyez localement correspond donc à ce qui sera exécuté après le déploiement.
{{< img src="images/PPCodeApp/image7.png" >}}

## Étape 6 : Déployer
 
Une fois l'application prête, `pac code push` (ou la commande de déploiement équivalente selon la version de votre CLI) la publie dans votre environnement Power Platform. Elle devient alors disponible comme n'importe quelle autre application Power Platform et est soumise aux mêmes règles de partage, de sécurité et de prévention contre la perte de données (DLP) que le reste de votre environnement.

**Dans le prochain article, nous découvrirons quelques fondamentaux de React et verrons comment afficher le profil de l'utilisateur actuel dans une Power Apps Code App avec React.**

## Pour aller plus loin
- [Présentation des Power Apps Code Apps — Microsoft Learn](https://learn.microsoft.com/power-apps/developer/code-apps/overview)
- [Démarrage rapide : créer une Code App à partir de zéro — Microsoft Learn](https://learn.microsoft.com/power-apps/developer/code-apps/how-to/create-an-app-from-scratch)
