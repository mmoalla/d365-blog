---
title: "Le framework SysOperation en X++ : traitements par lots, paramètres, requêtes et validation"
date: 2026-11-11
draft: false
categories: ["X++ & Dev"]
tags: ["X++", "SysOperation", "Batch", "Data Contract", "Validation"]
summary: "Le framework SysOperation est la méthode standard pour créer dans D365 F&O des processus exécutables en mode interactif ou par lots. Découvrez le rôle du contrat, du service et du contrôleur, ainsi que l'ajout de paramètres, d'une requête, de validations et de l'exécution par lots."
lightbox:
  enabled: true
justified_gallery:
  enabled: true
---

# Contexte

Chaque fois qu'un processus dans D365 F&O doit afficher une boîte de dialogue, s'exécuter en arrière-plan ou être planifié comme traitement récurrent, le **framework SysOperation** est la solution à privilégier. Il remplace l'ancien framework RunBase, et les recommandations de Microsoft pour les applications Finance et opérations sont de l'utiliser dès qu'un traitement doit effectuer des opérations ou des calculs.

Son principe fondamental consiste à répartir clairement les responsabilités entre quelques classes :

| Élément | Rôle |
|---|---|
| **Contrat de données** | Définit les paramètres du processus et l'emplacement de leur validation. La boîte de dialogue est générée automatiquement à partir de celui-ci |
| **Service** | Contient la logique métier proprement dite — le code qui effectue le travail |
| **Contrôleur** | Relie le contrat au service et détermine le mode d'exécution de l'opération (interactif, par lots, etc.) |
| **Générateur d'interface** *(facultatif)* | Personnalise la boîte de dialogue générée lorsque le comportement par défaut ne suffit pas |

Comme chaque élément est une classe classique, il reste concis et testable — et, point important dans D365, extensible via Chain of Command et les gestionnaires d'événements, ce qui était beaucoup plus difficile avec RunBase.

# Prérequis

- Un environnement de développement Dynamics 365 F&O avec Visual Studio
- Un modèle personnalisé pour héberger vos classes
- Une connaissance de base des classes et attributs X++

# Étapes
## Le contrat de données — vos paramètres

Le contrat est une classe marquée avec `[DataContract]`. Chaque paramètre est une variable privée exposée par une **méthode parm** décorée avec `[DataMember]`.

```xpp
[DataContract]
class MyCustomerProcessContract implements SysOperationValidatable
{
    CustGroupId custGroupId;
    TransDate   cutoffDate;
    NoYes       includeInactive;

    [
        DataMember,
        SysOperationLabel("Groupe de clients"),
        SysOperationDisplayOrder('1')
    ]
    public CustGroupId parmCustGroupId(CustGroupId _custGroupId = custGroupId)
    {
        custGroupId = _custGroupId;
        return custGroupId;
    }

    [
        DataMember,
        SysOperationLabel("Date limite"),
        SysOperationDisplayOrder('2')
    ]
    public TransDate parmCutoffDate(TransDate _cutoffDate = cutoffDate)
    {
        cutoffDate = _cutoffDate;
        return cutoffDate;
    }

    [
        DataMember,
        SysOperationLabel("Inclure les clients inactifs"),
        SysOperationDisplayOrder('3')
    ]
    public NoYes parmIncludeInactive(NoYes _includeInactive = includeInactive)
    {
        includeInactive = _includeInactive;
        return includeInactive;
    }
}
```

**Rôle de chaque attribut :**
- `[DataContract]` / `[DataMember]` — indiquent au framework que cette classe contient des paramètres et précisent quelles méthodes parm correspondent à ces paramètres. Seules les méthodes parm décorées avec `[DataMember]` apparaissent dans la boîte de dialogue et sont enregistrées avec le traitement par lots.
- `SysOperationLabel` — définit le libellé affiché dans la boîte de dialogue (dans le code réel, utilisez une référence à un libellé plutôt qu'une chaîne codée en dur).
- `SysOperationDisplayOrder` — détermine l'ordre des champs dans la boîte de dialogue.

Quelques autres attributs sont utiles à connaître : `SysOperationGroup` et `SysOperationGroupMember` organisent les paramètres en groupes nommés dans la boîte de dialogue, tandis que `SysOperationControlVisibility(false)` masque un paramètre dans l'interface tout en le conservant dans le contrat — pratique pour les valeurs transmises par programmation.

**Valeurs par défaut :** pour initialiser les paramètres, implémentez `SysOperationInitializable` et renseignez-les dans sa méthode `initialize()`. L'ajout de `[SysOperationAlwaysInitialize]` au contrat oblige le framework à appeler cette méthode à chaque fois, et pas uniquement lorsqu'aucune dernière valeur n'a été enregistrée.

## Le service — votre logique métier

```xpp
class MyCustomerProcessService extends SysOperationServiceBase
{
    [SysEntryPointAttribute(false)]
    public void process(MyCustomerProcessContract _contract)
    {
        CustGroupId custGroupId = _contract.parmCustGroupId();
        TransDate   cutoffDate  = _contract.parmCutoffDate();

        CustTable custTable;

        while select custTable
            where custTable.CustGroup == custGroupId
        {
            // ... le traitement métier s'effectue ici
            info(strFmt("Client %1 traité jusqu'au %2", custTable.AccountNum, cutoffDate));
        }
    }
}
```

L'opération du service reçoit le contrat en paramètre : c'est ainsi que les valeurs saisies par l'utilisateur dans la boîte de dialogue (ou enregistrées avec le traitement par lots) parviennent à votre code. Gardez cette classe centrée sur *l'exécution du travail* : pas de logique de boîte de dialogue ni de mode d'exécution. Ces responsabilités appartiennent précisément aux autres éléments.

## Le contrôleur — assembler les éléments

```xpp
class MyCustomerProcessController extends SysOperationServiceController
{
    protected void new()
    {
        super(
            classStr(MyCustomerProcessService),
            methodStr(MyCustomerProcessService, process),
            SysOperationExecutionMode::Synchronous);
    }

    public ClassDescription defaultCaption()
    {
        return "Traiter les clients";
    }

    public static MyCustomerProcessController construct(
        SysOperationExecutionMode _executionMode = SysOperationExecutionMode::Synchronous)
    {
        MyCustomerProcessController controller = new MyCustomerProcessController();
        controller.parmExecutionMode(_executionMode);
        return controller;
    }

    public static void main(Args _args)
    {
        MyCustomerProcessController controller = MyCustomerProcessController::construct();
        controller.parmArgs(_args);
        controller.startOperation();
    }
}
```

La méthode `new()` du contrôleur indique au framework **quelle classe de service et quelle méthode** exécuter. `main()` est le point d'entrée appelé par un élément de menu : il crée le contrôleur, lui transmet les arguments, puis `startOperation()` affiche la boîte de dialogue et lance l'opération.

Pour rendre le processus accessible aux utilisateurs, créez un **élément de menu d'action** dont le type d'objet est *Class* et dont l'objet est le contrôleur (`MyCustomerProcessController`).

## Exécution par lots

Comme le framework génère la boîte de dialogue, la prise en charge des traitements par lots est presque automatique : la boîte de dialogue générée comporte un onglet **Batch** et l'option standard **Run in the background**. Les utilisateurs peuvent ainsi lancer le même processus de façon interactive ou le planifier comme traitement récurrent, sans code supplémentaire.

Le comportement dépend du **mode d'exécution** :

| Mode | Comportement |
|---|---|
| `Synchronous` | S'exécute immédiatement ; l'utilisateur attend la fin du traitement |
| `Asynchronous` | S'exécute immédiatement sans bloquer la session de l'utilisateur |
| `ReliableAsynchronous` | S'exécute de manière asynchrone via l'infrastructure de traitements par lots, avec des garanties de fiabilité |
| `ScheduledBatch` | Passe toujours par le framework de traitements par lots |

Vous pouvez obliger un processus à toujours s'exécuter par lots en créant le contrôleur avec `ScheduledBatch`, ou laisser l'utilisateur choisir dans la boîte de dialogue.

**Créer des tâches par lots par programmation** — utile lorsqu'un traitement par lots doit lancer plusieurs étapes :

```xpp
MyCustomerProcessController controller = MyCustomerProcessController::construct();
controller.parmShowDialog(false);
controller.parmLoadFromSysLastValue(false);

batchHeader.addTask(controller);
```

`parmShowDialog(false)` ferme la boîte de dialogue (les paramètres sont alors définis dans le code), tandis que `parmLoadFromSysLastValue(false)` empêche le contrôleur de recharger les dernières valeurs saisies par l'utilisateur. Si le formulaire des tâches par lots doit permettre aux utilisateurs de modifier les paramètres de chaque tâche, décorez le contrat avec `[SysOperationJournaledParametersAttribute(true)]`.

## Ajouter un paramètre de requête

Il est souvent utile qu'un traitement par lots permette à l'utilisateur de filtrer *les enregistrements à traiter*, à l'aide de la même boîte de dialogue de filtre avancé que dans le reste de l'application. Une requête est ajoutée au contrat sous forme de chaîne empaquetée :

```xpp
[DataContract]
class MyCustomerProcessContract implements SysOperationValidatable
{
    str packedQuery;

    [
        DataMember,
        AifQueryType('_packedQuery', queryStr(CustTableQuery))
    ]
    public str parmQuery(str _packedQuery = packedQuery)
    {
        packedQuery = _packedQuery;
        return packedQuery;
    }

    public Query getQuery()
    {
        return new Query(SysOperationHelper::base64Decode(packedQuery));
    }

    public void setQuery(Query _query)
    {
        packedQuery = SysOperationHelper::base64Encode(_query.pack());
    }
}
```

`AifQueryType` associe le paramètre à une requête AOT existante (remplacez `CustTableQuery` par la requête correspondant à votre table) et définit ainsi les tables et champs sur lesquels l'utilisateur peut appliquer des filtres dans la boîte de dialogue. Dans le service, récupérez le filtre saisi par l'utilisateur et exécutez la requête :

```xpp
public void process(MyCustomerProcessContract _contract)
{
    Query    query    = _contract.getQuery();
    QueryRun queryRun = new QueryRun(query);
    CustTable custTable;

    while (queryRun.next())
    {
        custTable = queryRun.get(tableNum(CustTable));
        // ... traiter l'enregistrement filtré
    }
}
```

Comme la requête est stockée sous forme **empaquetée** dans le contrat, elle accompagne le traitement par lots comme n'importe quel autre paramètre : chaque exécution réutilise le filtre défini par l'utilisateur lors de la planification du traitement.

## Validation

L'implémentation de `SysOperationValidatable` dans le contrat (comme ci-dessus) vous permet d'ajouter une méthode `validate()`. Le framework l'appelle lorsque l'utilisateur confirme la boîte de dialogue, **avant** de lancer l'opération ou de créer le traitement par lots : les entrées non valides ne parviennent donc jamais au service.

```xpp
public boolean validate()
{
    boolean ret = true;

    if (!custGroupId)
    {
        ret = checkFailed("Le groupe de clients est obligatoire.");
    }

    if (cutoffDate > DateTimeUtil::getToday(DateTimeUtil::getUserPreferredTimeZone()))
    {
        ret = checkFailed("La date limite ne peut pas être dans le futur.");
    }

    return ret;
}
```

`checkFailed` ajoute le message à l'Infolog **et** renvoie `false` — c'est le modèle utilisé dans toute l'application standard pour la validation. Notez que les deux vérifications sont exécutées même si la première échoue : l'utilisateur voit ainsi *toutes* les erreurs à la fois, au lieu de devoir les corriger une par une.

Placez ici, dans le contrat, les règles qui portent sur plusieurs champs. Évitez de valider dans le service : lorsqu'il s'exécute — parfois des heures plus tard dans un traitement par lots — il est trop tard pour demander à l'utilisateur de corriger les données.

## Personnaliser la boîte de dialogue générée

Pour la plupart des processus, la boîte de dialogue automatique suffit. Si vous avez besoin de plus de contrôle — par exemple, pour activer ou désactiver un champ selon la valeur d'un autre — vous pouvez utiliser votre propre classe de **générateur d'interface** (*UI builder*). Enregistrez-la sur le contrat avec `SysOperationContractProcessing(classStr(MyUIBuilder))` ; le générateur peut ajouter des contrôles, les modifier ou réagir à leurs événements. N'y recourez que lorsque des attributs tels que `SysOperationLabel`, `SysOperationGroup` et `SysOperationControlVisibility` ne suffisent pas.

## Pièges courants

- **Oublier `[DataMember]` sur une méthode parm** : le paramètre n'apparaîtra pas dans la boîte de dialogue et ne sera pas enregistré avec le traitement par lots.
- **Placer la logique métier dans le contrôleur** : son rôle se limite à relier les éléments et à choisir le mode d'exécution ; la logique qui s'y trouve est plus difficile à tester et à étendre que dans le service.
- **Valider dans le service plutôt que dans le contrat** : dans un scénario par lots, le service peut s'exécuter bien après le départ de l'utilisateur. La validation doit avoir lieu au moment de la soumission, dans le contrat.
- **Effectuer un traitement lourd sans tenir compte du mode par lots** : un processus qui traite beaucoup de données doit être conçu dès le départ pour les traitements par lots (idempotent, redémarrable et réparti en lots de taille raisonnable), plutôt que d'être adapté après coup.
- **Coder les libellés en dur** : les valeurs de `SysOperationLabel` doivent être des références à des libellés, et non des chaînes littérales, afin que la boîte de dialogue soit traduite comme le reste de l'application.
- **Supposer que le traitement par lots redemandera les paramètres** : les valeurs enregistrées avec le traitement seront utilisées à chaque exécution. Si l'utilisateur souhaite d'autres valeurs, il doit modifier le traitement au lieu de compter sur l'affichage d'une nouvelle boîte de dialogue.

## Pour aller plus loin

- [Frameworks des applications Finance et opérations — Microsoft Learn](https://learn.microsoft.com/training/modules/design-deployment-finance-operations/8-frameworks)