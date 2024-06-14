# Play ID
**Play ID** is a new platform that empowers your apps with user authentication, analytics, cloud saves, leaderboards and achievements.

## Authentication
**Play ID Authentication** is OAuth 2.0 web service for Unity. The main goal is to simplify user authentication to `1 call`, with no need to perform complicated setup for each platform. Just redirect users to the main sign-in screen where they can sign in with **Google**, **Apple**, **Facebook**, **X**, **Twitter**, **Telegram** and **Microsoft**. Your app will receive user data using **deep linking** (when possible) or with an additional web request.
### User data disclosure
| Data / Platform | Google | Apple | Facebook | X (Twitter) | Telegram | Microsoft | VK |
| --- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Name | ⬤ | ⬤ | ⬤ | ⬤ | ⬤ | ⬤ | ⬤ |
| Email | ⬤ | ⬤ | ⬤ | @ | @ | ⬤ | ⬤ |
| Access token | ⬤ | ⬤ | ⬤ | ⬤ |  | ⬤ | ⬤ |
| Refresh token | ⬤ | ⬤ | ⬤ |  |  | ⬤ |  |
| ID Token (JWT) | ⬤ | ⬤ | ⬤ |  |  | ⬤ |  |

@ = `username` is returned instead of `email`

## Cloud Saves
**Play ID Cloud Saves** is cloud storage for saved games and other user data. The main goal is seamless user experience for cross platform apps:
- switch between different devices: mobile phone, tablet and PC
- restore app data when a device was broken / lost / stolen

Data size limit is 4096 bytes (1 record per 1 user per 1 app). Not designed for storing user generated content.

## Analytics, Leaderboards, Achievements
Under development.

## Unity plugin
**Play ID Plugin for Unity** is available on [Unity Asset Store](http://u3d.as/3j37). It provides integration with our web service.
 
## API reference

### Initialize 
```csharp
var playIdAuth = new PlayIdAuth();
```

### Sign-in 
```csharp
public void SignIn(Action<bool, string, UserInfo> callback, Platform platforms = Platform.Any, bool caching = true)
```

#### Arguments
- `callback` returns `success`, `error` and `userInfo`.
- `platforms` specifies available platforms on the sign-in screen. `Platform` enumeration has `[Flags]` attribute and can be treated as a bit field (that is, a set of flags). When the only platform is specified, the user is redirected directly to the platform sign-in page (platform selection is skipped).
- `caching` specifies if a previous sign-in result should be returned (if exists), otherwise the user will be redirected to the sign-in screen.

#### Example
```csharp
playIdAuth.SignIn(OnSignIn);
playIdAuth.SignIn(OnSignIn, platforms: Platform.Google | Platform.Apple | Platform.Facebook, caching: false);

private void OnSignIn(bool success, string error, UserInfo user)
{
    Debug.Log(success ? user.Name : error);
}
```

### Access token
Play ID `access token` is returned as a part of `TokenResponse`. By default, `access token` is valid for 7200 seconds.
```csharp
var accessToken = playIdAuth.SavedAuth.TokenResponse.AccessToken;
```

### Refresh access token
When Play ID `access token` is expired, you can request a new on.
```csharp
public void RefreshAccessToken(Action<bool, string, TokenResponse> callback)
```
#### Example
```csharp
playIdAuth.RefreshAccessToken(OnRefreshAccessToken);

void OnRefreshAccessToken(bool success, string error, TokenResponse tokenResponse)
{
    Debug.Log(success ? tokenResponse.AccessToken : error);
}
```

### Revoke access token
When Play ID `access token` is no longer needed, you can revoke it.
```csharp
public void SignOut(bool revokeAccessToken) // playId.SavedAuth will be deleted.
public void RevokeAccessToken(string accessToken) // playId.SavedAuth will be kept.
```
#### Example
```csharp
playIdAuth.SignOut(revokeAccessToken: true);
playIdAuth.RevokeAccessToken(playIdAuth.SavedAuth.TokenResponse.AccessToken);
```

### Internal data
In rare cases you may want to receive platform specific data (we call it `internal`). For example, user info JSON originally returned by Google, or Google access tokens or ID tokens (don't confuse it with Play ID tokens).
```csharp
public void RequestUserInfoForPlatform(Platform platform, Action<bool, string, string> callback)
public void RequestAccessTokenForPlatform(Platform platform, Action<bool, string, string> callback)
public void RequestIdTokenForPlatform(Platform platform, Action<bool, string, string> callback)
```
Before these calls:
- ensure that user is signed in with Play ID by checking `playIdAuth.SavedAuth != null`;
- ensure that user is authorized on the selected platform with `playIdAuth.SavedAuth.UserInfo.Platforms.HasFlag(Platform.Google)`;
- check if Play ID access token is expired `playIdAuth.SavedAuth.TokenResponse.Expired`;
- if Play ID access token is expired, call `playIdAuth.RefreshAccessToken`;
#### Example
```csharp
// Ensure that the user is signed in with the selected platform and the access token is not expired.
playIdAuth.RequestUserInfoForPlatform(Platform.Google, OnGetUserInfo);

void OnGetUserInfo(bool success, string error, string userInfo)
{
    Debug.Log(success ? userInfo : error);
}
```

### Linking accounts
You can link multiple accounts to one user. For example, if the user is already signed in with Google, you may also want to link his Apple of Facebook account. To do this, you can navigate users to the sign-in screen. Platforms where the user is already signed in will be hidden. Optionally, you can specify which platforms will be available for linking.
```csharp
public void Link(Action<bool, string, UserInfo> callback, Platform platforms = Platform.Any)
```
#### Example
```csharp
playIdAuth.Link(OnLinkPlatform);            

void OnLinkPlatform(bool success, string error, UserInfo user)
{
    Debug.Log(success ? $"{user.Platforms}" : error);
}
```

### Uninking accounts
You can unlink a platform account from a user. For example, if the user is already signed in with both Google and Apple, and you want to unlink his Apple account.
```csharp
public void Unlink(Action<bool, string, UserInfo> callback, Platform platform)
```

### Account deletion
Users can delete their accounts by visiting https://playid.org/auth/delete. This will result complete Play ID account deletion including all data related to other apps: linked accounts, saved games, leaderboards, achievements. You can make a button inside your app that will execute the following code:
```csharp
Application.OpenURL("https://playid.org/auth/delete");
```

### Cloud saves
To make calls, you need to create an instance of `CloudSaves` class with a valid access token. Data size limit is 4096 bytes (1 record per 1 user per 1 app).
```csharp
public void Save(string data, Action<bool, string> callback)
public void Save(byte[] data, Action<bool, string> callback)
public void Load(Action<bool, string, byte[]> callback)
public void LoadString(Action<bool, string, string> callback)
```
#### Examples
```csharp
// Ensure that the user is signed in and the access token is not expired.
var accessToken = playIdAuth.SavedAuth.TokenResponse.AccessToken;
var cloudSaves = new CloudSaves(accessToken);
var data = new { progress = 10, timestamp = DateTime.UtcNow };
var json = JsonConvert.SerializeObject(data);

cloudSaves.Save(json, OnSave);
cloudSaves.LoadString(OnLoad);

void OnSave(bool success, string error)
{
    Debug.Log(success ? "Saved!" : error);
}

void OnLoad(bool success, string error, string data)
{
    Debug.Log(success ? data : error);
}
```

## For users (players)
- [Terms of use](https://github.com/hippogamesunity/PlayID/wiki/Terms-of-use)
- [Privacy policy](https://github.com/hippogamesunity/PlayID/wiki/Privacy-policy)
