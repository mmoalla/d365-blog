---
title: "Power Apps Code Apps — Partie 2 : afficher le profil de l'utilisateur connecté avec React"
date: 2026-09-08
draft: false
categories: ["Power Platform"]
tags: ["Power Apps", "Code Apps", "React", "TypeScript", "Office 365 Users"]
summary: "Dans la Partie 1, on a créé le squelette d'une Code App et connecté le connecteur Office 365 Users. Ici, on construit le composant React qui affiche le nom, le poste et la photo de l'utilisateur actuellement connecté."
series: "Power Apps Code Apps"
---
 
## Contexte
 
Dans la [Partie 1 de cette série]({{% ref path="posts/PPCodeApp/powerappscodeapp" lang="fr" %}}), on a créé le squelette d'une Code App avec le template officiel, puis ajouté le connecteur **Office 365 Users** via :
 
```bash
pac code add-data-source -a "shared_office365users" -c "<connector_id>"
```

Cette commande a généré des fichiers TypeScript typés dans `src/generated/` - un modèle (`Office365UsersModel.ts`) et un service (`Office365UsersService.ts`) - mais elle ne construit **aucune interface**. À ce stade, l'application affiche encore le template Vite par défaut (le compteur de clics et les logos React/Vite).
 
Dans cet article, on va :
- Voir rapidement quelques fondamentaux React utilisés (useState, useEffect, useMemo, hook personnalisé).
- Écrire un hook qui récupère le profil et la photo de l'utilisateur connecté.
- Construire un composant qui les affiche proprement, avec gestion du chargement et des erreurs.
## Prérequis
 
- Le projet de la Partie 1, avec le connecteur Office 365 Users déjà ajouté.
- Les notions de base du framework ReactJS.

## Étape 1 : Notions fondamentaux de ReactJS
 
Si vous découvrez React, voici les trois briques qu'on va utiliser, en une phrase chacune :
 
- **`useState`** - est utilisé pour stocker des données dans un composant. Cela permet de mettre à jour l'interface utilisateur de manière dynamique. Lorsque l'état change, le composant est automatiquement re-rendu.

- **`useEffect`** - est utilisé pour effectuer des effets de bord dans React. Il exécute du code après le rendu du composant. Il est principalement utilisé pour les appels API et les minuteurs. Il permet également de gérer les écouteurs d'événements et les fonctions de nettoyage.

- **`useMemo`** - est utilisé pour améliorer les performances. Il stocke des valeurs calculées en mémoire. Il évite les recalculs inutiles. Il ne s'exécute que lorsque les dépendances changent. Il est utile pour les calculs lourds et le filtrage de données.

- **Hook personnalisé** — une fonction dont le nom commence par `use`, qui regroupe de la logique réutilisable (state + effect ensemble) pour qu'un composant n'ait pas à la réécrire à chaque fois
C'est exactement cette combinaison qu'on retrouve dans le hook ci-dessous.
 
## Étape 2 : Creation du hook `userProfile`

1 - Créez un dossier *hooks* dans `src`.
2 - Créez le fichier `src/hooks/useUserProfile.ts` :
 
```ts
import { useEffect,useState, useMemo } from 'react';
import { Office365UsersService } from '../generated/services/Office365UsersService';
import type { User } from '../generated/models/Office365UsersModel';

export function useUserProfile() {
    const [user, setUser] = useState<User | null>(null);
    const [photo, setPhoto] = useState<string | null>(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState<string | null>(null);

    const fetchUserProfile = async () => {
        try {
            setLoading(true);
            setError(null);

            const myProfile = await Office365UsersService.MyProfile();

            if(myProfile.data) {
                setUser(myProfile.data);
                try {
                    const userPhoto = await Office365UsersService.UserPhoto(myProfile.data.Id);
                    if(userPhoto.data) {
                        setPhoto(userPhoto.data);
                    }
                    else
                    {
                        setError('No photo data returned for user ID:' + myProfile.data.Id);
                    }
                } catch (photoError) {
                    setError(photoError instanceof Error ? photoError.message : 'An error occurred while fetching user photo');
                }
            }
            else {
                setError('No profile data returned');
            }
        } 
        catch (err) {
            setError(err instanceof Error ? err.message : 'An error occurred while fetching user profile');
        } 
        finally {
            setLoading(false);
        }
    };

    // useEffect s'exécute une seule fois au montage du composant
    useEffect(() => {
        fetchUserProfile();
    }, []); 

    // useMemo pour mémoriser l'objet retourné et éviter des re-renders inutiles
    const result = useMemo(() => {
        return { user, photo, loading, error };
    }, [user, photo, loading, error]);

    return result;
}
```
 
**Ce qui se passe, étape par étape :**
- `Office365UsersService.MyProfile()` appelle la méthode `MyProfile` du connecteur - elle retourne le profil de l'utilisateur **actuellement connecté**.
- Une fois le profil récupéré, `Office365UsersService.UserPhoto(myProfile.data.Id)` va chercher sa photo, dans un second appel séparé - les deux opérations sont distinctes côté connecteur.
- Les erreurs de la photo sont capturées **sans faire échouer tout le hook** : un utilisateur sans photo de profil doit quand même pouvoir voir son nom et son poste.

## Étape 3 : Création d'un composant.
1- Créez le dossier *components* dans `src`.
2- Créez le composant `src/components/UserProfile.tsx` :
 
```tsx
import { useUserProfile } from '../../hooks/useUserProfile';

function UserProfile() {
    const { user, photo, loading, error } = useUserProfile();

    if(loading) {
        return <>
            <progress value={undefined} />
        </>;
    }

    if(error) {
        return <div>Error: {error}</div>;
    }

    return (
        <div style={{ display: 'flex', alignItems: 'center', gap: '16px' }}>
            {photo ? (
                <img
                    src={`data:image/jpeg;base64,${photo}`}
                    alt={user?.DisplayName}
                    style={{ width: '64px', height: '64px', borderRadius: '50%' }}
                />
            ) : (
                <div style={{ width: '64px', height: '64px', borderRadius: '50%', background: '#ccc' }} />
            )}
            <div>
                <strong>{user?.DisplayName}</strong>
                <p style={{ margin: 0 }}>{user?.UserPrincipalName}</p>
            </div>
        </div>
    );
}

export default UserProfile;
```
 
**Le point technique à retenir ici :** le champ `photo` renvoyé par le connecteur est une chaîne encodée en base64, **sans préfixe**. Pour qu'un navigateur l'affiche comme une image, il faut reconstruire une URI de données complète : `data:image/jpeg;base64,` suivi du contenu — c'est un piège fréquent, l'image reste invisible si ce préfixe est oublié.
 
## Étape 4 : Intégration du composant dans `App.tsx`
 
Dans le composant App.tsx, remplace le contenu du template par défaut :
 
```tsx
import './App.css'
import UserProfile from './components/UserProfile/UserProfile.tsx'

function App() {
  return (
    <>
      <h1>My code App</h1>
      <UserProfile />
    </>
  )
}

export default App
```
 
Lance `npm run dev` - la page doit maintenant afficher votre photo, votre nom et votre adresse mail, tirés en temps réel de votre profil Office 365, à la place du compteur de démonstration.
Lancez l'application en cliquant sur l'url.
{{< img src="images/PPCodeApp/image8.png" >}}

Le nom d'utilisateur, la photo ainsi que l'email s'affichent.
{{< img src="images/PPCodeApp/image9.png" >}}

## Pièges courants
 
- **Oublier le préfixe `data:image/jpeg;base64,`** sur la photo : l'appel réussit, la donnée arrive bien, mais l'image reste cassée dans le navigateur puisque `<img src="...">` attend une URI complète, pas juste le contenu encodé.
- **Ne pas gérer l'absence de photo séparément de l'absence de profil** : un utilisateur sans photo configurée dans Microsoft 365 est un cas normal, pas une erreur bloquante — le hook capture volontairement cette erreur sans casser l'affichage du reste du profil.
- **Oublier `useMemo`** sur la valeur de retour du hook : sans lui, chaque re-rendu du composant recrée un nouvel objet `{ user, photo, loading, error }`, ce qui peut déclencher des re-rendus inutiles en cascade si ce hook est utilisé par plusieurs composants.
- **Ne pas relancer `pac code add-data-source`** après avoir ajouté le connecteur si les fichiers `generated/` ne sont pas présents dans le projet — sans eux, `Office365UsersService` n'existe simplement pas encore.
## Pour aller plus loin
 
- [Référence du connecteur Office 365 Users — Microsoft Learn](https://learn.microsoft.com/connectors/office365users/)
- [Partie 1 de cette série : créer et déployer une Code App]({{% ref path="posts/PPCodeApp/powerappscodeapp" lang="fr" %}})