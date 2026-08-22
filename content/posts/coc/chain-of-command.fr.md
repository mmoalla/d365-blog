---
title: "Chaîne de commande sur les classes, tables, formulaires et entités de données : le guide complet"
date: 2026-07-13
draft: false
categories: ["X++ & Dev"]
tags: ["X++", "Chain of Command", "Extensions"]
summary: "La Chaîne de commande n'est pas réservée aux classes et tables : les formulaires, sources de données, champs, contrôles et entités de données ont chacun leur propre syntaxe d'extension. Voici un tour complet, avec un exemple pour chaque cas."
---

## Contexte

**La Chaîne de commande (Chain of Command ou CoC)**, aux côtés des Event Handlers, est l'un des deux principaux mécanismes d'extensibilité dans Dynamics 365 F&O. Contrairement aux Event Handlers, la CoC vous permet de **modifier le comportement** d'une méthode existante : altérer sa valeur de retour, transformer ses paramètres, ou même bloquer son exécution entièrement.

La syntaxe repose sur trois éléments :
- Une classe `final`, marquée avec l'attribut `[ExtensionOf(...)]` qui indique quel objet elle étend.
- Une méthode qui reproduit **exactement** la signature de la méthode originale (même nom, mêmes paramètres, même type de retour).
- Le mot-clé `next`, qui appelle la méthode originale (ou l'extension précédente dans la chaîne) — l'équivalent de `base()` en C#

```xpp
[ExtensionOf(classStr(SalesFormLetter))]
final class PrefixCompanyCompanySalesFormLetter_Extension
{
    public void update(SalesFormLetter_Invoice _formLetter)
    {
        // Logique avant l'exécution originale
        next update(_formLetter);
        // Logique après l'exécution originale
    }
}
```

Ce qui n'est pas souvent couvert, c'est que **chaque type d'objet X++ a sa propre fonction de ciblage** à l'intérieur de l'attribut `[ExtensionOf(...)]`. Utiliser la mauvaise fonction pour le mauvais objet est l'erreur la plus courante lors de la découverte de la CoC sur les formulaires — la compilation échoue souvent sans un message très clair sur la cause réelle.

Voici la fonction à utiliser pour chacun des sept points d'extension les plus courants :

| Objet à étendre | Fonction de ciblage |
|---|---|
| Classe | `classStr(ClassName)` |
| Table | `tableStr(TableName)` |
| Formulaire (méthodes sur le formulaire lui-même) | `formStr(FormName)` |
| Source de données du formulaire | `formDataSourceStr(FormName, DataSourceName)` |
| Champ de données sur un formulaire | `formDataFieldStr(FormName, DataSourceName, FieldName)` |
| Contrôle de formulaire | `formControlStr(FormName, ControlName)` |
| Entité de données | `tableStr(DataEntityName)` (une entité de données est techniquement une table) |

## Prérequis

- Dynamics 365 F&O version 10.0.x.
- Visual Studio avec les outils de développement D365 F&O.
- Un modèle personnalisé existant.
- Être familiarisé avec les bases de la Chaîne de commande (recommandé).

## 1. Étendre une classe

Le principe est celui que nous venons de voir dans l'introduction. Un autre exemple courant : intercepter un calcul de montant dans une classe métier standard.

```xpp
[ExtensionOf(classStr(CustPaymSchedRule))]
final class CustPaymSchedRule_PrefixCompanyCompany_Class_Extension
{
    public AmountMST calcPaymAmount(AmountMST _amount, int _numberOfPayments)
    {
        AmountMST amount = next calcPaymAmount(_amount, _numberOfPayments);

        if (this.RoundUp)
        {
            amount = PrefixCompanyCompanyRoundingHelper::roundUp(amount);
        }

        return amount;
    }
}
```

## 2. Étendre une table

```xpp
[ExtensionOf(tableStr(CustTable))]
final class CustTable_PrefixCompanyCompany_T_Extension
{
    void initValue()
    {
        next initValue();

        //définir votre code personnalisé
    }
}
```

## 3. Étendre un formulaire

Ici, nous ciblons le formulaire lui-même — utile pour accrocher ses méthodes globales comme `init()` :

```xpp
[ExtensionOf(formStr(CustTable))]
final class CustTable_PrefixCompany_F_Extension
{
    public void init()
    {
        next init();

        //votre code de personnalisation va ici
    }
}
```

## 4. Étendre une source de données du formulaire

Chaque `FormDataSource` (le lien entre un formulaire et une table) peut être étendu indépendamment — utile pour accrocher le chargement des enregistrements, la validation, ou l'activation :

```xpp
[ExtensionOf(formDataSourceStr(CustTable, CustTable))]
final class CustTable_PrefixCompany_DS_Extension
{
    public int active()
    {
        int ret;
        ret = next active();

        CustTable custTableLocal = this.cursor();

        //En fonction de l'enregistrement actuellement sélectionné,
        //activez/désactivez ou affichez/masquez votre bouton ou contrôle ici.

        return ret;
    }
}
```

**Remarque importante :** le paramètre `FormDataSourceName` dans `formDataSourceStr(FormName, FormDataSourceName)` fait référence au **nom de la source de données dans l'arborescence du concepteur du formulaire**, pas nécessairement au nom de la table sous-jacente — les deux sont souvent identiques, mais pas toujours (une source de données peut être renommée dans le concepteur).

## 5. Étendre un champ de données sur un formulaire

```xpp
[ExtensionOf(formDataFieldStr(CustTable, CustTable, CustGroup)]
final class CustTable_PrefixCompany_DF_Extension
{
    public void modified()
    {
        FormDataObject formDataObject = any2Object(this) as FormDataObject;
        FormDataSource formDataSource = formDataObject.datasource();
        CustTable custTable;

        next modified();

        custTable = formDataSource.cursor();
        custTable.CustGroup = CustGroup::find(custTable.CustGroup).CustGroup;

    }
}
```

## 6. Étendre un contrôle de formulaire

```xpp
[ExtensionOf(formControlStr(CustTable, ButtonDelete))]
final class CustTable_PrefixCompany_FC_Extension
{
    public void clicked()
    {
        FormControl formButtonControl = any2Object(this) as FormControl;
        FormDataSource formDatasource = formButtonControl.formRun().dataSource(tableStr(CustTable));
        CustTable custTable = formDatasource.cursor();

        next clicked();

        info(strFmt("Cust Account %1", custTable.AccountNum));
    }
}
```

## 7. Étendre une entité de données

```xpp
[ExtensionOf(tableStr(CustomersV3Entity))]
final class CustomersV3Entity_PrefixCompany_Entity_Extension
{
    public boolean validateWrite()
    {
        boolean ret = next validateWrite();

        //Code personnalisé
        if(CustTable.CustGroup != 'Group1')
        {
            ret = checkFailed("message");
        }

        return ret;
    }
}
```

C'est particulièrement utile pour ajouter une logique de validation ou de transformation lors des importations de données via le Data Management Framework, sans toucher à la définition standard de l'entité.

## Pièges courants

- **Oublier que le nom de la source de données peut différer du nom de la table** : vérifiez toujours le nom exact affiché dans l'arborescence du concepteur du formulaire, pas seulement le nom de la table.
- **Ne pas gérer le cas où `next` n'est jamais appelé sur un contrôle** : comme dans l'exemple du bouton "Supprimer", c'est intentionnel et légitime, mais assurez-vous que c'est réellement voulu et non une omission — autrement, cliquer sur un bouton standard peut sembler "ne rien faire" sans un message d'erreur clair pour l'utilisateur.
- **Étendre une entité de données en pensant qu'elle se comporte différemment d'une table** : ce n'est pas le cas — elle suit exactement les mêmes règles de CoC qu'une table standard.

## Lectures complémentaires

- [Documentation Microsoft Learn : method wrapping et Chain of Command](https://learn.microsoft.com/dynamics365/fin-ops-core/dev-itpro/extensibility/method-wrapping-coc)