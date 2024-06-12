# PlayID
Simple OAuth 2.0 sign-in for Unity.

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
