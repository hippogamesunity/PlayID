# Play ID
**Play ID** is a new platform that empowers your apps with user authentication, analytics, cloud saves, leaderboards and achievements.

## Authentication
**Play ID Authentication** is OAuth 2.0 web service for Unity. The main goal is to simplify user authentication to `1 call`, with no need to perform complicated setup for each platform. Just redirect users to the main sign-in screen where they can sign in with **Google**, **Apple**, **Facebook**, **X**, **Twitter**, **Telegram** and **Microsoft**. Your app will receive user data using **deep linking** (when possible) or with an additional web request.
## Analytics, Cloud saves, Leaderboards, Achievements
Under development.

## Unity plugin
**Play ID Plugin for Unity** is available on **Unity Asset Store**. It provides integration with our web service.
 
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
- `platforms` specifies available platforms on the sign-in screen. `Platform` enumeration has `[Flags]` attribute and can be treated as a bit field (that is, a set of flags).
- `caching` specifies if a previous sign-in result should be returned (if exists), otherwise the user will be redirected to the sign-in screen.
#### Examples
```csharp
playIdAuth.SignIn(OnSignIn);
PlayIdAuth.SignIn(OnSignIn, platforms: Platform.Google | Platform.Apple | Platform.Facebook, caching: false);

private void OnSignIn(bool success, string error, UserInfo user)
{
    Debug.Log(success ? user.Name : error);
}
```
