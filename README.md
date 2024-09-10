# Play ID
**Play ID** is a new social platform that empowers your apps with user Authentication, Remote Config, Cloud Saves, Analytics, Leaderboards and Achievements.

## Authentication
**Play ID Authentication** is OAuth 2.0 web service for Unity. The main goal is to simplify user authentication to `1 call`, with no need to perform complicated setup for each platform. Just redirect users to the main sign-in screen where they can sign in with **Google**, **Apple**, **Facebook**, **X**, **Twitter**, **Telegram** and **Microsoft**. Your app will receive user data using **Deep Linking** (when possible) or with an additional web request.
### User data disclosure
| Data / Platform | Google | Apple | Facebook | X (Twitter) | Telegram | Microsoft | VK |
| --- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Name | ⬤ | ⬤ | ⬤ | ⬤ | ⬤ | ⬤ | ⬤ |
| Email | ⬤ | ⬤ | ⬤ | @ | @ | ⬤ | ⬤ |
| Access token | ◍ | ◍ | ◍ | ◍ |  | ◍ | ◍ |
| Refresh token | ◍ | ◍ | ◍ |  |  | ◍ |  |
| ID token (JWT) | ⬤ | ⬤ | ⬤ |  |  | ⬤ |  |

@ - `username` is returned instead of `email`

◍ - we don't expose access tokens at the moment (feel free to request and describe your use cases)

## Remote Config
**Play ID Remote Config** provides your apps with remote configuration that can be requested on app start. Recommended format is JSON.
- adjust game difficulty on fly
- create events and challanges
- adjust service functions (for example, advertisement priority and intervals)

Data size limit is 1024 bytes.

## Cloud Saves
**Play ID Cloud Saves** is cloud storage for saved games and other user data. The main goal is seamless user experience for cross platform apps:
- switch between different devices: mobile phone, tablet and PC
- restore app data when a device was broken / lost / stolen

Data size limit is 4096 bytes (1 record per 1 user per 1 app). Not designed for storing user generated content.

## Leaderboards
**Play Leaderboards** provides your apps with online scores. Competition is a good way to increase retention.

## Analytics, Achievements
Under development.

## Unity plugin
**Play ID Plugin for Unity** is available on [Unity Asset Store](http://u3d.as/3j37). It provides integration with our web service.

## Setup steps
1. Install the plugin using Unity Package Manager
2. Create a new developer account and a new app
   - Visit [Register as Play ID developer](https://playid.org/auth/dev) and sign in
   - When creating a new Play ID developer account, the FREE plan is activated for your app (please visit [Plans](https://github.com/hippogamesunity/PlayID/wiki/Plans) for more information)
   - Get `SecretKey` and `ClientId` from the output
   - If you're already a Play ID developer, your configuration will be displayed
   - If you need to register one more app, please email your request to `hippogamesunity@gmail.com`
3. Come up with an unique `Redirect URI Scheme` that will be used to activate your app with **Deep Linking** (for example: `myapp`)
4. Return to Unity and open `PlayId/AppSettings` scriptable object
   - Set `Secret Key` and `Client Id`
   - Clear `RedirectUriWhitelist` and add `[REDIRECT_URI_SCHEME]://oauth2/playid` to `RedirectUriWhitelist` (for example: `myapp://oauth2/playid`)
   - Set `Name`, `Icon` and `RemoteConfig` (optional)
   - Press `Save`
5. Open `PlayId/Resources/AuthSettings` scriptable object, set `Client Id` and `Redirect URI Scheme`
6. Setup **Deep Linking**
   - For **Android**: add `<data android:scheme="[REDIRECT_URI_SCHEME]" />` at the end of `<intent-filter>` of your AndroidManifest.xml (for example: `<data android:scheme="myapp" />`).
   - For **iOS and macOS**: navigate to `Player Settings > Other > Configuration` and add your `Redirect URI Scheme` to `Supported URL schemes`. In Xcode, make sure that the URL scheme is added ([Register your URL scheme](https://developer.apple.com/documentation/xcode/defining-a-custom-url-scheme-for-your-app#Register-your-URL-scheme)).
   - For **Universal Windows Platform**: navigate to `Player Settings > Publishing Settings` and set `Protocol` as your `Redirect URI Scheme`, then enable `InternetClient` in `Capabilities`.
   - For **Windows**: refer to [Settings for Windows](https://github.com/hippogamesunity/PlayID/wiki/Settings-for-Windows) section.
8. Run `Examples` scene and test sign-in

## API reference

### Initialize
To start using Play ID, create an instance of `PlayIdServices`. The constructor will load `AuthSettings` from Resources. Alternatively, you can use `PlayIdServices.Instance` singleton.
```csharp
var playIdServices = new PlayIdServices();
```

### Sign-in
If the user is not signed in, it will be redirected to Play ID sign-in page using a default system browser.
If the user was previously signed in, the method will return `User` (the access token will be refreshed automatically if expired).
```csharp
public void SignIn(Action<bool, string, User> callback, Platform platforms = Platform.Any, bool caching = true)
```

#### Arguments
- `callback` returns `success`, `error` and `user`.
- `platforms` specifies available platforms on the sign-in screen. `Platform` enumeration has `[Flags]` attribute and can be treated as a bit field (that is, a set of flags). When the only platform is specified, the user is redirected directly to the platform sign-in page (platform selection is skipped).
- `caching` specifies if a previous sign-in result should be returned (if exists) without requesting new user data. If the access token is expired, it will be refreshed anyway.

#### Example
```csharp
playId.Auth.SignIn(OnSignIn);
playId.Auth.SignIn(OnSignIn, platforms: Platform.Google | Platform.Apple | Platform.Facebook, caching: false);

private void OnSignIn(bool success, string error, User user)
{
    Debug.Log(success ? user.Name : error);
}
```

### ID token validation
`ID token` (JWT) is returned as a part of `TokenResponse`. Please refer to [ID token validation](https://github.com/hippogamesunity/PlayID/wiki/ID-token-validation) section.

### Access token
Play ID `access token` is returned as a part of `TokenResponse`. By default, `access token` is valid for 7200 seconds. The access token is requred to make Play ID API calls (Bearer Authorization).
```csharp
var accessToken = playId.Auth.SavedAuth.TokenResponse.AccessToken;
```

### Refresh access token
When Play ID `access token` is expired, you can refresh it with `refresh token`.
```csharp
public void RefreshAccessToken(Action<bool, string, TokenResponse> callback)
```
#### Example
```csharp
playId.Auth.RefreshAccessToken(OnRefreshAccessToken);

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
playId.Auth.SignOut(revokeAccessToken: true);
playId.Auth.RevokeAccessToken(playId.Auth.SavedAuth.TokenResponse.AccessToken);
```

### Internal data
In rare cases you may want to receive platform specific data with `Internals` service available for authorized users (`user.Internals`). For example, you can get user info JSON originally returned by Google or ID tokens / JWT (don't confuse it with Play ID tokens).
| Method | Arguments | Description |
| :--- | :--- | :--- |
| RequestUserInfoForPlatform | Platform platform, Action<bool, string, string> callback | Returns original user info for the selected platform. It may contain additional information about the user. |
| RequestIdTokenForPlatform | Platform platform, bool refresh, Action<bool, string, string> callback | Returns an original ID token (JWT) for the selected platform. For Google and Apple only: if ID token is expired and `refresh` is TRUE, it will be refreshed automatically. |

Before these calls:
- ensure that user is signed in with Play ID by checking `playId.Auth.SavedAuth != null`;
- ensure that user is authorized on the selected platform with `play.IdAuth.SavedAuth.UserInfo.Platforms.HasFlag(Platform.Google)`;
- check if Play ID access token is expired `playId.Auth.SavedAuth.TokenResponse.Expired`;
- if Play ID access token is expired, call `playId.Auth.RefreshAccessToken`;
#### Example
```csharp
// Ensure that the user is signed in with the selected platform and the access token is not expired.
user.Internals.RequestUserInfoForPlatform(Platform.Google, OnGetUserInfo);

void OnGetUserInfo(bool success, string error, string userInfo)
{
    Debug.Log(success ? userInfo : error);
}

user.Internals.RequestIdTokenForPlatform(Platform.Apple, OnGetIdToken);

void OnGetIdToken(bool success, string error, string idToken)
{
    Debug.Log(success ? idToken : error);
}
```

### Linking accounts
You can link multiple accounts to one user. For example, if the user is already signed in with Google, you may also want to link his Apple of Facebook account. To do this, you can navigate users to the sign-in screen. Platforms where the user is already signed in will be hidden. Optionally, you can specify which platforms will be available for linking.
```csharp
public void Link(Action<bool, string, UserInfo> callback, Platform platforms = Platform.Any)
```
#### Example
```csharp
playId.Auth.Link(OnLinkPlatform);            

void OnLinkPlatform(bool success, string error, User user)
{
    Debug.Log(success ? $"{user.Platforms}" : error);
}
```

### Uninking accounts
You can unlink a platform account from a user. For example, if the user is already signed in with both Google and Apple, and you want to unlink his Apple account.
```csharp
public void Unlink(Action<bool, string, User> callback, Platform platform)
```

### Account deletion
Users can delete their accounts by visiting https://playid.org/auth/delete. This will result complete Play ID account deletion including all data related to other apps: linked accounts, saved games, leaderboards, achievements. You can make a button inside your app that will execute the following code:
```csharp
Application.OpenURL("https://playid.org/auth/delete");
```

### Cloud Saves
To use this API, the user should be signed in and have a valid access token. Data size limit is 4096 bytes (1 record per 1 user per 1 app).
When the user performs `SignIn`, `CloudSaves` instance becomes available as a part of `User` object and is accessible with `user.CloudSaves`. Another option is create a new instance of `CloudSaves` and passing a valid access token to its' constructor (available as `playId.SavedAuth.TokenResponse.AccessToken`).
```csharp
public void Save(string data, Action<bool, string> callback)
public void Save(byte[] data, Action<bool, string> callback)
public void Load(Action<bool, string, byte[]> callback)
public void LoadString(Action<bool, string, string> callback)
```
#### Examples
```csharp
// Ensure that the user is signed in and the access token is not expired.
var data = new { progress = 10, timestamp = DateTime.UtcNow };
var json = JsonConvert.SerializeObject(data);

user.CloudSaves.Save(json, OnSave);
user.CloudSaves.LoadString(OnLoad);

void OnSave(bool success, string error)
{
    Debug.Log(success ? "Saved!" : error);
}

void OnLoad(bool success, string error, string data)
{
    Debug.Log(success ? data : error);
}
```

### Leaderboards
To start using this API, you should create leaderboards. Just open your AppSettings (Scriptable Object), set `Leaderboards` (IDs) and press `Save`.
To report scores, users should be authorized. `Leaderboards` instance is available as a part of `User` object and is accessible with `user.Leaderboards`.
The are 2 ways to load scores:
1. If the user is not authorized, you can call `Leaderboards.LoadScores` (static method)
2. If the user is authorized, you can call `user.Leaderboards.LoadScores_`
```csharp
public void ReportScore(string leaderboardId, long score, Action<bool, string> callback)
public void LoadScores_(string leaderboardId, int top, int period, List<int> friends, Action<bool, string, List<Score>> callback)
public static void LoadScores(string leaderboardId, int top, int period, List<int> scope, Action<bool, string, List<Score>> callback)
```
#### Examples
```csharp
user.Leaderboards.ReportScore("gold", Random.Range(0, 10000), OnReport);

void OnReport(bool success, string error)
{
    Output.text = success ? "Score reported!" : error;
}

var scope = new List<int>(); // A list of users that should be guaranteed included to scores.

Scripts.Services.Leaderboards.LoadScores("gold", 10, 90, scope, OnReportScore);

void OnLoadScores(bool success, string error, List<Score> scores)
{
    Output.text = success ? string.Join('\n', scores.Select(i => $"#{i.Position} - {i.UserName} - {i.Value}")) : error;
}
```

### Remote Config
To start using this API, you should create configuration. Just open your AppSettings (Scriptable Object), set `Remote Config` and press `Save`.
`RemoteConfig` instance is available as a part of `PlayIdServices` instance.
```csharp
public void Load(Action<bool, string, string> callback)
```
#### Examples
```csharp
PlayIdServices.Instance.RemoteConfig.Load(OnLoadRemoteConfig);

void OnLoadRemoteConfig(bool success, string error, string remoteConfig)
{
    Output.text = success ? remoteConfig : error;
}
```

## Integration with other services
- [Integration with Unity Authentication Service](https://github.com/hippogamesunity/PlayID/wiki/Integration-with-Unity-Authentication-Service)

## For users (players)
- [Terms of use](https://github.com/hippogamesunity/PlayID/wiki/Terms-of-use)
- [Privacy policy](https://github.com/hippogamesunity/PlayID/wiki/Privacy-policy)
