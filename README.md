# Play ID
**Play ID** is a lightweight, high-performance Backend-as-a-Service (BaaS) tool built by game developers, for game developers. Say goodbye to heavy, native social media SDKs that bloat your build size and complicate server deployment. 

Integrate authentication, remote config, сloud saves, analytics, leaderboards and achievements in under 5 minutes. At no service cost!

---

## 🚀 Quick Start

### 1. Installation
Install the package from [Unity Asset Store](https://assetstore.unity.com/packages/slug/287608).

### 2. Initialization
* Open `AppSettings` scriptable object, press `Developer configuration`, sign in and obtain `Client Id`
* Open `AuthSettings` scriptable object and set `Client Id`
* Create create a new script and add it to your scene

```csharp
using UnityEngine;
using PlayID;

public class AutoSignIn : MonoBehaviour
{
    private void Start()
    {
        PlayIdServices.Instance.Auth.SignIn(OnSignIn);
    }

    private static void OnSignIn(bool success, string error, User user)
    {
        Debug.Log(success ? $"Hello, {user.Name}!" : error);
    }
}
```
For more details visit [Setup-steps](https://github.com/hippogamesunity/PlayID/wiki/Setup-steps).

---

## 🛠 Architectural Diagrams

### 1. Authentication Flow (Zero Native SDKs)
Authentication runs entirely through the system browser and Deep Linking. This ensures a 0 MB increase to your APK/IPA file size and complete security for user tokens.

```text
 [ Game (Unity) ] ──────────────────( 1. Click "Sign in" )─────────────────► [ Device Browser ]
        ▲                                                                        │
        │                                                                ( 2. Select Provider )
        │                                                                        │
        │                                                                        ▼
 ( 5. Deep Link + JWT )                                                  [ OAuth Provider Sign-In ]
        │                                                                        │
        │                                                                ( 3. Authorize App )
        │                                                                        │
        │                                                                        ▼
 [ PlayIdCore SDK ] ◄────( 4. Exchange Codes )──── [ Play ID Server ] ◄──────────┘
```

## 📚 Documentation Sections

* [Services description](https://github.com/hippogamesunity/PlayID/wiki)
* [Setup steps](https://github.com/hippogamesunity/PlayID/wiki/Setup-steps)
* [API reference](https://github.com/hippogamesunity/PlayID/wiki/API-reference)
* [Troubleshooting](https://github.com/hippogamesunity/PlayID/wiki/Troubleshooting)
* [Privacy policy](https://github.com/hippogamesunity/PlayID/wiki/Privacy-policy)
