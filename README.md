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

## Analytics, Cloud Saves, Leaderboards, Achievements
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
public void SignOut(bool revokeAccessToken = false) // playId.SavedAuth will be deleted.
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
if (playIdAuth.SavedAuth == null)
{
    Debug.Log("Please sign in first.");
}
else if (!playIdAuth.SavedAuth.UserInfo.Platforms.HasFlag(Platform.Google))
{
    Debug.Log("User was not authorized by Google.");
}
else if (playIdAuth.SavedAuth.TokenResponse.Expired)
{
    Debug.Log("Access token expired.");
}
else
{
    playIdAuth.RequestUserInfoForPlatform(Platform.Google, OnGetUserInfo);
}

void OnGetUserInfo(bool success, string error, string userInfo)
{
    Debug.Log(success ? userInfo : error);
}
```

### Account deletion
Users can delete their accounts by visiting https://playid.org/auth/delete. This will result complete Play ID account deletion including all data related to other apps: linked accounts, saved games, leaderboards, achievements. You can make a button inside your app that will execute the following code:
```csharp
Application.OpenURL("https://playid.org/auth/delete");
```

### For users (players)
- [Terms of use](https://github.com/hippogamesunity/PlayID/wiki/Terms-of-use)
- [Privacy policy](https://github.com/hippogamesunity/PlayID/wiki/Privacy-policy)
