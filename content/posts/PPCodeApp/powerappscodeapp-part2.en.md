---
title: "Power Apps Code Apps — Part 2: Displaying the connected user profile with React"
date: 2026-09-08
draft: false
categories: ["Power Platform"]
tags: ["Power Apps", "Code Apps", "React", "TypeScript", "Office 365 Users"]
summary: "In Part 1, we created the skeleton of a Code App and connected the Office 365 Users connector. Here, we build the React component that displays the name, job title, and photo of the currently connected user."
series: "Power Apps Code Apps"
---
 
## Context
 
In the [Part 1 of this series]({{% ref path="posts/PPCodeApp/powerappscodeapp" lang="en" %}}), we created the skeleton of a Code App with the official template and then added the **Office 365 Users** connector via:
 
```bash
pac code add-data-source -a "shared_office365users" -c "<connector_id>"
```

This command generated typed TypeScript files in `src/generated/` — a model (`Office365UsersModel.ts`) and a service (`Office365UsersService.ts`) — but it does not build any UI. At this point, the application still shows the default Vite template (the click counter and the React/Vite logos).
 
In this article, we will:
- Quickly review a few React fundamentals used here (`useState`, `useEffect`, `useMemo`, custom hook).
- Write a hook that retrieves the connected user's profile and photo.
- Build a component that displays them cleanly, including loading and error handling.

## Prerequisites
 
- The project from Part 1, with the Office 365 Users connector already added.
- Basic knowledge of the ReactJS framework.

## Step 1: React fundamentals
 
If you are new to React, here are the three building blocks we will use, each in one sentence:
 
- **`useState`** — used to store data in a component. This allows the UI to update dynamically. When state changes, the component is automatically re-rendered.

- **`useEffect`** — used to perform side effects in React, executes code after the component renders. It is mainly used for API calls and timers. It also handles event listeners and cleanup functions.

- **`useMemo`** - Used to improve performance, stores calculated values in memory, prevents unnecessary recalculations. It runs only when dependencies change. It is useful for heavy calculations and filtering data.

- **Custom hook** — a function whose name starts with `use`, grouping reusable logic (state + effect together) so a component does not have to rewrite it every time.
This is exactly the combination used in the hook below.
 
## Step 2: Creating the `userProfile` hook

1 - Create a `hooks` folder in `src`.
2 - Create the file `src/hooks/useUserProfile.ts`:
 
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

    // useEffect runs only once when the component mounts
    useEffect(() => {
        fetchUserProfile();
    }, []); 

    // useMemo to memoize the returned object and avoid unnecessary re-renders
    const result = useMemo(() => {
        return { user, photo, loading, error };
    }, [user, photo, loading, error]);

    return result;
}
```
 
**What happens step by step:**
- `Office365UsersService.MyProfile()` calls the connector method `MyProfile` — it returns the profile of the **currently connected user**.
- Once the profile is retrieved, `Office365UsersService.UserPhoto(myProfile.data.Id)` fetches the user's photo in a second separate call — the two operations are distinct on the connector side.
- Photo errors are captured **without failing the whole hook**: a user without a profile photo should still be able to see their name and title.

## Step 3: Creating a component
1 - Create the `components` folder in `src`.
2 - Create the component `src/components/UserProfile.tsx`:
 
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
 
**The key technical point to remember:** the `photo` field returned by the connector is a base64-encoded string, **without a prefix**. For a browser to display it as an image, you must rebuild a full data URI: `data:image/jpeg;base64,` followed by the content — this is a common trap, because the image remains invisible if the prefix is missing.
 
## Step 4: Integrating the component into `App.tsx`
 
In the `App.tsx` component, replace the default template content:
 
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
 
Run `npm run dev` — the page should now display your photo, name, and email, pulled in real time from your Office 365 profile, replacing the demo counter.
Open the application by clicking the URL.
{{< img src="images/PPCodeApp/image8.png" >}}

The username, photo, and email are displayed.
{{< img src="images/PPCodeApp/image9.png" >}}

## Common pitfalls
 
- **Forgetting the `data:image/jpeg;base64,` prefix** on the photo: the call succeeds and the data arrives correctly, but the image stays broken in the browser because `<img src="...">` expects a full URI, not just the encoded content.
- **Not handling the absence of a photo separately from the absence of a profile**: a user without a configured profile photo in Microsoft 365 is a normal case, not a blocking error — the hook intentionally captures this without breaking the rest of the profile display.
- **Forgetting `useMemo`** on the hook return value: without it, each re-render creates a new object like `{ user, photo, loading, error }`, which can trigger unnecessary cascading re-renders if this hook is used by multiple components.
- **Not re-running `pac code add-data-source`** after adding the connector if the generated `generated/` files are not present in the project — without them, `Office365UsersService` simply does not exist yet.
 
## Going further
 
- [Office 365 Users connector reference — Microsoft Learn](https://learn.microsoft.com/connectors/office365users/)
- [Part 1 of this series: create and deploy a Code App](/posts/power-apps-code-apps-overview/)
