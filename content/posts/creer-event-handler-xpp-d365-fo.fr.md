---
title: "Créer un Event Handler en X++ dans Dynamics 365 F&O"
date: 2026-07-22
draft: false
categories: ["X++ & Dev"]
tags: ["X++", "Event Handler", "Extensions", "Chain of Command"]
summary: "Comment intercepter proprement un événement standard dans D365 F&O sans toucher au code natif, avec un exemple concret sur la table SalesTable."
---

## Contexte
Dans Dynamics 365 Finance & Operations, il est interdit de modifier directement le code standard Microsoft (le modèle "over-layering" a disparu depuis AX 2012). Toute personnalisation doit passer par des **extensions**.

Deux mécanismes principaux existent pour intervenir sur du code existant :

- **Chain of Command (CoC)** : Permet d’**étendre ou de modifier le comportement d’une méthode existante** sans la remplacer.
- **Event Handlers** : Servent à répondre à des événements déclenchés par le système ou par l’utilisateur, comme les clics sur un bouton, les modifications de champ ou les mises à jour d’enregistrements, sans toucher à l’objet de base.

 **Utilise un Event Handler lorsque:** tu veux exécuter une logique à l’occurrence d’un événement. Tu souhaites éviter de modifier le code existant. C’est particulièrement adapté aux événements liés à **l’interface, aux données et à la validation.**

## Prérequis
- Dynamics 365 F&O version 10.0.x (Tier 1 dev box ou cloud-hosted environment ou UDE)
- Visual Studio.

## Étapes
### 1. Créer le projet et la classe d'extension
Dans Visual Studio, crée un nouveau projet D365 F&O, puis ajoute une classe d'événement :

```xpp
public class CustTableHandler
{
}
```

### 2. Identifier l'événement à intercepter
Ouvre la table `CustTable` dans l'Application Explorer et repère la méthode `confirmAndSaveCustGroupChange()`. On veut ici exécuter une action **après** la création d'une commande de vente.

{{< img src="images/eventHandler/SalesTable.png" alt="Sales table object" >}}
{{< img src="images/eventHandler/SalesCopyEvent.png" alt="Sales table object" >}}

Deux options d'événements existent pour la plupart des méthodes standard :
- `[PostHandlerFor]` — s'exécute après la méthode originale
- `[PreHandlerFor]` — s'exécute avant la méthode originale

### 3. Écrire le Post Event Handler
```xpp
class CustTableHandler
{
    [PreHandlerFor(tableStr(CustTable), tableStaticMethodStr(CustTable, confirmAndSaveCustGroupChange))]
    public static void CustTable_Post_confirmAndSaveCustGroupChange(XppPrePostArgs args)
    {
        //Object instance of the CustTable table
        CustTable custTable = args.getThis();
        //parameter value in confirmAndSaveCustGroupChange method
        boolean isCustGroupSetOnce = args.getArg('isCustGroupSetOnce');
        //return value of the confirmAndSaveCustGroupChange method
        boolean valueModifiedReturn = args.getReturnValue();
    }
}
```

**Points clés :**
- `args.getThis()` récupère l'instance de l'objet (ici la ligne `SalesTable` qui vient d'être insérée)
- `args.getArg('isCustGroupSetOnce')` récupère le paramètre de la méthode.
- `args.getReturnValue()` récupère la valeur de retour de la methode (à utiliser en cas de post event handler).
- Le handler doit être `static` et prendre un paramètre `XppPrePostArgs`
- Le nom de la méthode n'a pas d'importance fonctionnelle, mais la convention `Table_Post_methode` facilite la lecture

### 4. Ajouter une validation avec un Pre Event Handler
Pour bloquer une opération avant qu'elle ne se produise (ex: empêcher la création d'une commande sans client actif) :

```xpp
[PreHandlerFor(tableStr(SalesTable), tableMethodStr(SalesTable, insert))]
public static void SalesTable_Pre_insert(XppPrePostArgs args)
{
    SalesTable salesTable = args.getThis();
    CustTable custTable = CustTable::find(salesTable.CustAccount);

    if (custTable.Blocked != CustVendorBlocked::No)
    {
        throw error(strFmt("Le compte client %1 est bloqué.", salesTable.CustAccount));
    }
}
```

## Pièges courants
- **Event handlers multiples sur le même événement** : l'ordre d'exécution entre extensions de différents modèles n'est pas garanti, évite les dépendances d'ordre

## Pour aller plus loin
- Documentation Microsoft : <a href="https://learn.microsoft.com/fr-fr/dynamics365/fin-ops-core/dev-itpro/dev-ref/xpp-events" target="_blank">Event Handeler</a> 
- Prochain article : "Chain of Command en X++ : quand l'utiliser plutôt qu'un Event Handler"
