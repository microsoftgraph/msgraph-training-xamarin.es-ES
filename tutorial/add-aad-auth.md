<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD. Esto es necesario para obtener el token de acceso de OAuth necesario para llamar a Microsoft Graph. En este paso, integrará la [biblioteca de autenticación de Microsoft para .net (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) en la aplicación.

En el **Explorador de soluciones**, expanda el proyecto **GraphTutorial** y haga clic con el botón secundario en la carpeta **Models** . Elija **Agregar clase >...** Asigne un nombre `OAuthSettings` a la clase y elija **Agregar**. Abra el archivo **OAuthSettings.CS** y reemplace el contenido por lo siguiente.

```cs
namespace GraphTutorial.Models
{
    public static class OAuthSettings
    {
        public const string ApplicationId = "YOUR_APP_ID_HERE";
        public const string RedirectUri = "YOUR_REDIRECT_URI_HERE";
        public const string Scopes = "User.Read Calendars.Read";
    }
}
```

> [!IMPORTANT]
> Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el `OAuthSettings.cs` archivo del control de código fuente para evitar la pérdida inadvertida del identificador de la aplicación.

## <a name="implement-sign-in"></a>Implementar el inicio de sesión

Abra el archivo **app.Xaml.CS** en el proyecto **GraphTutorial** y agregue las siguientes `using` instrucciones en la parte superior del archivo.

```cs
using GraphTutorial.Models;
using Microsoft.Identity.Client;
using Microsoft.Graph;
using System.Diagnostics;
using System.Linq;
using System.Net.Http.Headers;
```

Agregue las siguientes propiedades a la `App` clase.

```cs
// UIParent used by Android version of the app
public static UIParent AuthUIParent = null;

// Microsoft Authentication client for native/mobile apps
public static PublicClientApplication PCA;

// Microsoft Graph client
public static GraphServiceClient GraphClient;
```

A continuación, cree un `PublicClientApplication` nuevo en el constructor de `App` la clase.

```cs
public App()
{
    InitializeComponent();

    PCA = new PublicClientApplication(OAuthSettings.ApplicationId);

    MainPage = new MainPage();
}
```

Ahora, actualice `SignIn` la función para que `PublicClientApplication` use el para obtener un token de acceso. Agregue el siguiente código encima de `await GetUserInfo();` la línea.

```cs
var scopes = OAuthSettings.Scopes.Split(' ');

// First, attempt silent sign in
// If the user's information is already in the app's cache,
// they won't have to sign in again.
try
{
    var accounts = await PCA.GetAccountsAsync();
    var silentAuthResult = await PCA.AcquireTokenSilentAsync(
        scopes, accounts.FirstOrDefault());

    Debug.WriteLine("User already signed in.");
    Debug.WriteLine($"Access token: {silentAuthResult.AccessToken}");
}
catch (MsalUiRequiredException)
{
    // This exception is thrown when an interactive sign-in is required.
    // Prompt the user to sign-in
    var authResult = await PCA.AcquireTokenAsync(scopes, AuthUIParent);
    Debug.WriteLine($"Access Token: {authResult.AccessToken}");
}
```

Este código primero intenta obtener un token de acceso en modo silencioso. Si la información de un usuario ya se encuentra en la memoria caché de la aplicación (por ejemplo, si el usuario ha cerrado la aplicación anteriormente sin cerrar sesión), esto se realizará correctamente y no hay motivo para preguntar al usuario. Si no hay información de un usuario en la memoria caché, la `AcquireTokenSilentAsync` función inicia una `MsalUiRequiredException`. En este caso, el código llama a la función interactiva para obtener un `AcquireTokenAsync`token,.

Ahora, actualice `SignOut` la función para quitar la información del usuario de la memoria caché. Agregue el código siguiente al principio de la `SignOut` función.

```cs
// Get all cached accounts for the app
// (Should only be one)
var accounts = await PCA.GetAccountsAsync();
while (accounts.Any())
{
    // Remove the account info from the cache
    await PCA.RemoveAsync(accounts.First());
    accounts = await PCA.GetAccountsAsync();
}
```

Por último, actualice `MainPage` la clase para iniciar sesión cuando se cargue. Agregue la siguiente función a la `MainPage` clase en **mainpage.Xaml.CS**.

```cs
protected override async void OnAppearing()
{
    base.OnAppearing();
    await (Application.Current as App).SignIn();
}
```

### <a name="update-android-project-to-enable-sign-in"></a>Actualizar el proyecto de Android para habilitar el inicio de sesión

Cuando se usa en un proyecto de Xamarin Android, la biblioteca de autenticación de Microsoft tiene algunos [requisitos específicos de Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).

En primer lugar, necesita agregar el URI de redireccionamiento desde el registro de la aplicación al manifiesto de Android. Para ello, agregue una actividad nueva al proyecto **GraphTutorial. Android** . Haga clic con el botón derecho en **GraphTutorial. Android** y elija **Agregar**y, a continuación, **nuevo elemento.**... Elija **actividad**, asigne un nombre `MsalActivity`a la actividad y elija **Agregar**.

Abra el archivo **MsalActivity.CS** y elimine `[Activity(Label = "MsalActivity")]` la línea y, a continuación, agregue los siguientes atributos sobre la declaración de clase.

```cs
// This class only exists to create the necessary activity in the Android
// manifest. Doing it this way allows the value of the RedirectUri constant
// to be inserted at build.
[Activity(Name = "microsoft.identity.client.BrowserTabActivity")]
[IntentFilter(new[] { Intent.ActionView },
    Categories = new[] { Intent.CategoryDefault, Intent.CategoryBrowsable },
    DataScheme = Models.OAuthSettings.RedirectUri, DataHost = "auth")]
```

A continuación, Abra **MainActivity.CS** y agregue las `using` siguientes instrucciones en la parte superior del archivo.

```cs
using Microsoft.Identity.Client;
using Android.Content;
```

A continuación, reemplace `OnActivityResult` la función para pasar el control a la biblioteca de MSAL. Agregue lo siguiente a la `MainActivity` clase.

```cs
protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
{
    base.OnActivityResult(requestCode, resultCode, data);
    AuthenticationContinuationHelper
        .SetAuthenticationContinuationEventArgs(requestCode, resultCode, data);
}
```

### <a name="update-ios-project-to-enable-sign-in"></a>Actualizar el proyecto de iOS para habilitar el inicio de sesión

> [!IMPORTANT]
> Como MSAL requiere el uso de un archivo de derechos. plist, debe configurar Visual Studio con su cuenta de desarrollador de Apple para habilitar el aprovisionamiento. Para obtener más información, consulte [aprovisionamiento de dispositivos para Xamarin. iOS](/xamarin/ios/get-started/installation/device-provisioning).

Cuando se usa en un proyecto de Xamarin iOS, la biblioteca de autenticación de Microsoft tiene algunos [requisitos específicos de iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).

En primer lugar, debe habilitar el acceso a llaves. En el explorador de soluciones, expanda el proyecto **GraphTutorial. iOS** y, a continuación, abra el archivo contitles **. plist** . Busque el derecho de la **cadena de claves** y seleccione **Habilitar cadena de claves**. En **grupos de llaves**, agregue una entrada con el `com.YOUR_DOMAIN.GraphTutorial`formato.

![Captura de pantalla de la configuración de derechos de llaves](./images/enable-keychain-access.png)

A continuación, debe registrar el URI de redireccionamiento que configuró en el paso de registro de la aplicación como un tipo de dirección URL que controla la aplicación. Abra el archivo **info. plist** y realice los siguientes cambios.

- En la pestaña **aplicación** , compruebe que el valor del **identificador de lote** coincida con el valor que estableció para los grupos de **llaves** en **derechos. plist**. Si no es así, actualícelo ahora.
- En la ficha **avanzadas** , busque la sección **tipos de URL** . Agregue aquí un tipo de dirección URL con los valores siguientes:
    - **Identificador**: establezca en el valor del identificador del **lote**
    - **Esquemas de URL**: establezca el URI de redireccionamiento desde el registro de la aplicación que comienza con`msal`
    - **Rol**:`Editor`
    - **Icono**: dejar vacío

![Una captura de pantalla de la sección tipos de URL de info. plist](./images/add-url-type.png)

Por último, actualice el código en el proyecto **GraphTutorial. iOS** para controlar el redireccionamiento durante la autenticación. Abra el archivo **AppDelegate.CS** y agregue la siguiente `using` instrucción en la parte superior del archivo.

```cs
using Microsoft.Identity.Client;
```

Agregue la línea siguiente para `FinishedLaunching` que funcione justo antes `return` de la instrucción.

```cs
// Specify the Keychain access group
App.PCA.iOSKeychainSecurityGroup = "com.graphdevx.GraphTutorial";
```

Por último, reemplace `OpenUrl` la función para pasar la dirección URL a la biblioteca de MSAL. Agregue lo siguiente a la `AppDelegate` clase.

```cs
// Handling redirect URL
// See: https://github.com/azuread/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics
public override bool OpenUrl(UIApplication app, NSUrl url, NSDictionary options)
{
    AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(url);
    return true;
}
```

## <a name="storing-the-tokens"></a>Almacenamiento de tokens

Cuando se usa la biblioteca de autenticación de Microsoft en un proyecto Xamarin, se aprovecha de forma predeterminada el almacenamiento seguro nativo para almacenar en caché los tokens. Para obtener más información, vea Serialización de la [caché](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) de tokens.

## <a name="test-sign-in"></a>Inicio de sesión de prueba

En este punto, si ejecuta la aplicación y pulsa el botón **iniciar sesión** , se le pedirá que inicie sesión. Si el inicio de sesión se realiza correctamente, debería ver el token de acceso impreso en el resultado del depurador.

![Captura de pantalla de la ventana de resultados en Visual Studio](./images/debugger-access-token.png)

## <a name="get-user-details"></a>Obtener detalles del usuario

Ahora, actualice `SignIn` la función en **app.Xaml.CS** para inicializar el `GraphServiceClient`. Agregue el siguiente código antes de `await GetUserInfo();` la línea.

```cs
// Initialize Graph client
GraphClient = new GraphServiceClient(new DelegateAuthenticationProvider(
    async (requestMessage) =>
    {
        var accounts = await PCA.GetAccountsAsync();

        var result = await PCA.AcquireTokenSilentAsync(scopes, accounts.FirstOrDefault());

        requestMessage.Headers.Authorization =
            new AuthenticationHeaderValue("Bearer", result.AccessToken);
    }));
```

Ahora, actualice `GetUserInfo` la función para obtener los detalles del usuario de Microsoft Graph. Reemplace la función `GetUserInfo` existente por lo siguiente.

```cs
private async Task GetUserInfo()
{
    // Get the logged on user's profile (/me)
    var user = await GraphClient.Me.Request().GetAsync();

    UserPhoto = ImageSource.FromStream(() => GetUserPhoto());
    UserName = user.DisplayName;
    UserEmail = string.IsNullOrEmpty(user.Mail) ? user.UserPrincipalName : user.Mail;
}
```

Si guarda los cambios y ejecuta la aplicación ahora, después de iniciar sesión, la interfaz de usuario se actualizará con el nombre para mostrar y la dirección de correo electrónico del usuario.

