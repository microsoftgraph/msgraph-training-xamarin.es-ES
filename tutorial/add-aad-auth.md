<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="9b6e6-101">En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="9b6e6-102">Esto es necesario para obtener el token de acceso de OAuth necesario para llamar a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="9b6e6-103">En este paso, integrará la [biblioteca de autenticación de Microsoft para .net (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-103">In this step you will integrate the [Microsoft Authentication Library for .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) into the application.</span></span>

<span data-ttu-id="9b6e6-104">En el **Explorador de soluciones**, expanda el proyecto **GraphTutorial** y haga clic con el botón secundario en la carpeta **Models** .</span><span class="sxs-lookup"><span data-stu-id="9b6e6-104">In **Solution Explorer**, expand the **GraphTutorial** project and right-click the **Models** folder.</span></span> <span data-ttu-id="9b6e6-105">Elija **Agregar clase >...** Asigne un nombre `OAuthSettings` a la clase y elija **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-105">Choose **Add > Class...**. Name the class `OAuthSettings` and choose **Add**.</span></span> <span data-ttu-id="9b6e6-106">Abra el archivo **OAuthSettings.CS** y reemplace el contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-106">Open the **OAuthSettings.cs** file and replace its contents with the following.</span></span>

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
> <span data-ttu-id="9b6e6-107">Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el `OAuthSettings.cs` archivo del control de código fuente para evitar la pérdida inadvertida del identificador de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-107">If you're using source control such as git, now would be a good time to exclude the `OAuthSettings.cs` file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="9b6e6-108">Implementar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="9b6e6-108">Implement sign-in</span></span>

<span data-ttu-id="9b6e6-109">Abra el archivo **app.Xaml.CS** en el proyecto **GraphTutorial** y agregue las siguientes `using` instrucciones en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-109">Open the **App.xaml.cs** file in the **GraphTutorial** project, and add the following `using` statements to the top of the file.</span></span>

```cs
using GraphTutorial.Models;
using Microsoft.Identity.Client;
using Microsoft.Graph;
using System.Diagnostics;
using System.Linq;
using System.Net.Http.Headers;
```

<span data-ttu-id="9b6e6-110">Agregue las siguientes propiedades a la `App` clase.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-110">Add the following properties to the `App` class.</span></span>

```cs
// UIParent used by Android version of the app
public static UIParent AuthUIParent = null;

// Microsoft Authentication client for native/mobile apps
public static PublicClientApplication PCA;

// Microsoft Graph client
public static GraphServiceClient GraphClient;
```

<span data-ttu-id="9b6e6-111">A continuación, cree un `PublicClientApplication` nuevo en el constructor de `App` la clase.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-111">Next, create a new `PublicClientApplication` in the constructor of the `App` class.</span></span>

```cs
public App()
{
    InitializeComponent();

    PCA = new PublicClientApplication(OAuthSettings.ApplicationId);

    MainPage = new MainPage();
}
```

<span data-ttu-id="9b6e6-112">Ahora, actualice `SignIn` la función para que `PublicClientApplication` use el para obtener un token de acceso.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-112">Now update the `SignIn` function to use the `PublicClientApplication` to get an access token.</span></span> <span data-ttu-id="9b6e6-113">Agregue el siguiente código encima de `await GetUserInfo();` la línea.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-113">Add the following code above the `await GetUserInfo();` line.</span></span>

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

<span data-ttu-id="9b6e6-114">Este código primero intenta obtener un token de acceso en modo silencioso.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-114">This code first attempts to get an access token silently.</span></span> <span data-ttu-id="9b6e6-115">Si la información de un usuario ya se encuentra en la memoria caché de la aplicación (por ejemplo, si el usuario ha cerrado la aplicación anteriormente sin cerrar sesión), esto se realizará correctamente y no hay motivo para preguntar al usuario.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-115">If a user's information is already in the app's cache (for example, if the user closed the app previously without signing out), this will succeed, and there's no reason to prompt the user.</span></span> <span data-ttu-id="9b6e6-116">Si no hay información de un usuario en la memoria caché, la `AcquireTokenSilentAsync` función inicia una `MsalUiRequiredException`.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-116">If there is not a user's information in the cache, the `AcquireTokenSilentAsync` function throws an `MsalUiRequiredException`.</span></span> <span data-ttu-id="9b6e6-117">En este caso, el código llama a la función interactiva para obtener un `AcquireTokenAsync`token,.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-117">In this case, the code calls the interactive function to get a token, `AcquireTokenAsync`.</span></span>

<span data-ttu-id="9b6e6-118">Ahora, actualice `SignOut` la función para quitar la información del usuario de la memoria caché.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-118">Now update the `SignOut` function to remove the user's information from the cache.</span></span> <span data-ttu-id="9b6e6-119">Agregue el código siguiente al principio de la `SignOut` función.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-119">Add the following code to the beginning of the `SignOut` function.</span></span>

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

<span data-ttu-id="9b6e6-120">Por último, actualice `MainPage` la clase para iniciar sesión cuando se cargue.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-120">Finally, update the `MainPage` class to sign-in when it loads.</span></span> <span data-ttu-id="9b6e6-121">Agregue la siguiente función a la `MainPage` clase en **mainpage.Xaml.CS**.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-121">Add the following function to the `MainPage` class in **MainPage.xaml.cs**.</span></span>

```cs
protected override async void OnAppearing()
{
    base.OnAppearing();
    await (Application.Current as App).SignIn();
}
```

### <a name="update-android-project-to-enable-sign-in"></a><span data-ttu-id="9b6e6-122">Actualizar el proyecto de Android para habilitar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="9b6e6-122">Update Android project to enable sign-in</span></span>

<span data-ttu-id="9b6e6-123">Cuando se usa en un proyecto de Xamarin Android, la biblioteca de autenticación de Microsoft tiene algunos [requisitos específicos de Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).</span><span class="sxs-lookup"><span data-stu-id="9b6e6-123">When used in a Xamarin Android project, the Microsoft Authentication Library has a few [requirements specific to Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).</span></span>

<span data-ttu-id="9b6e6-124">En primer lugar, necesita agregar el URI de redireccionamiento desde el registro de la aplicación al manifiesto de Android.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-124">First, you need to add the redirect URI from your app registration to the Android manifest.</span></span> <span data-ttu-id="9b6e6-125">Para ello, agregue una actividad nueva al proyecto **GraphTutorial. Android** .</span><span class="sxs-lookup"><span data-stu-id="9b6e6-125">To do this, add a new activity to the **GraphTutorial.Android** project.</span></span> <span data-ttu-id="9b6e6-126">Haga clic con el botón derecho en **GraphTutorial. Android** y elija **Agregar**y, a continuación, **nuevo elemento.**... Elija **actividad**, asigne un nombre `MsalActivity`a la actividad y elija **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-126">Right-click the **GraphTutorial.Android** and choose **Add**, then **New Item...**. Choose **Activity**, name the activity `MsalActivity`, and choose **Add**.</span></span>

<span data-ttu-id="9b6e6-127">Abra el archivo **MsalActivity.CS** y elimine `[Activity(Label = "MsalActivity")]` la línea y, a continuación, agregue los siguientes atributos sobre la declaración de clase.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-127">Open the **MsalActivity.cs** file and delete the `[Activity(Label = "MsalActivity")]` line, then add the following attributes above the class declaration.</span></span>

```cs
// This class only exists to create the necessary activity in the Android
// manifest. Doing it this way allows the value of the RedirectUri constant
// to be inserted at build.
[Activity(Name = "microsoft.identity.client.BrowserTabActivity")]
[IntentFilter(new[] { Intent.ActionView },
    Categories = new[] { Intent.CategoryDefault, Intent.CategoryBrowsable },
    DataScheme = Models.OAuthSettings.RedirectUri, DataHost = "auth")]
```

<span data-ttu-id="9b6e6-128">A continuación, Abra **MainActivity.CS** y agregue las `using` siguientes instrucciones en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-128">Next, open **MainActivity.cs** and add the following `using` statements to the top of the file.</span></span>

```cs
using Microsoft.Identity.Client;
using Android.Content;
```

<span data-ttu-id="9b6e6-129">A continuación, reemplace `OnActivityResult` la función para pasar el control a la biblioteca de MSAL.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-129">Then, override the `OnActivityResult` function to pass control to the MSAL library.</span></span> <span data-ttu-id="9b6e6-130">Agregue lo siguiente a la `MainActivity` clase.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-130">Add the following to the `MainActivity` class.</span></span>

```cs
protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
{
    base.OnActivityResult(requestCode, resultCode, data);
    AuthenticationContinuationHelper
        .SetAuthenticationContinuationEventArgs(requestCode, resultCode, data);
}
```

### <a name="update-ios-project-to-enable-sign-in"></a><span data-ttu-id="9b6e6-131">Actualizar el proyecto de iOS para habilitar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="9b6e6-131">Update iOS project to enable sign-in</span></span>

> [!IMPORTANT]
> <span data-ttu-id="9b6e6-132">Como MSAL requiere el uso de un archivo de derechos. plist, debe configurar Visual Studio con su cuenta de desarrollador de Apple para habilitar el aprovisionamiento.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-132">Because MSAL requires use of an Entitlements.plist file, you must configure Visual Studio with your Apple developer account to enable provisioning.</span></span> <span data-ttu-id="9b6e6-133">Para obtener más información, consulte [aprovisionamiento de dispositivos para Xamarin. iOS](/xamarin/ios/get-started/installation/device-provisioning).</span><span class="sxs-lookup"><span data-stu-id="9b6e6-133">For more information, see [Device provisioning for Xamarin.iOS](/xamarin/ios/get-started/installation/device-provisioning).</span></span>

<span data-ttu-id="9b6e6-134">Cuando se usa en un proyecto de Xamarin iOS, la biblioteca de autenticación de Microsoft tiene algunos [requisitos específicos de iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).</span><span class="sxs-lookup"><span data-stu-id="9b6e6-134">When used in a Xamarin iOS project, the Microsoft Authentication Library has a few [requirements specific to iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).</span></span>

<span data-ttu-id="9b6e6-135">En primer lugar, debe habilitar el acceso a llaves.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-135">First, you need to enable Keychain access.</span></span> <span data-ttu-id="9b6e6-136">En el explorador de soluciones, expanda el proyecto **GraphTutorial. iOS** y, a continuación, abra el archivo contitles **. plist** .</span><span class="sxs-lookup"><span data-stu-id="9b6e6-136">In Solution Explorer, expand the **GraphTutorial.iOS** project, then open the **Entitlements.plist** file.</span></span> <span data-ttu-id="9b6e6-137">Busque el derecho de la **cadena de claves** y seleccione **Habilitar cadena de claves**.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-137">Locate the **Keychain** entitlement, and select **Enable KeyChain**.</span></span> <span data-ttu-id="9b6e6-138">En **grupos de llaves**, agregue una entrada con el `com.YOUR_DOMAIN.GraphTutorial`formato.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-138">In **Keychain Groups**, add an entry with the format `com.YOUR_DOMAIN.GraphTutorial`.</span></span>

![Captura de pantalla de la configuración de derechos de llaves](./images/enable-keychain-access.png)

<span data-ttu-id="9b6e6-140">A continuación, debe registrar el URI de redireccionamiento que configuró en el paso de registro de la aplicación como un tipo de dirección URL que controla la aplicación.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-140">Next, you need to register the redirect URI you configured in the app registration step as a URL type that your app handles.</span></span> <span data-ttu-id="9b6e6-141">Abra el archivo **info. plist** y realice los siguientes cambios.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-141">Open the **Info.plist** file and make the following changes.</span></span>

- <span data-ttu-id="9b6e6-142">En la pestaña **aplicación** , compruebe que el valor del **identificador de lote** coincida con el valor que estableció para los grupos de **llaves** en **derechos. plist**.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-142">On the **Application** tab, check that the value of **Bundle identifier** matches the value you set for **Keychain Groups** in **Entitlements.plist**.</span></span> <span data-ttu-id="9b6e6-143">Si no es así, actualícelo ahora.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-143">If it doesn't, update it now.</span></span>
- <span data-ttu-id="9b6e6-144">En la ficha **avanzadas** , busque la sección **tipos de URL** .</span><span class="sxs-lookup"><span data-stu-id="9b6e6-144">On the **Advanced** tab, locate the **URL Types** section.</span></span> <span data-ttu-id="9b6e6-145">Agregue aquí un tipo de dirección URL con los valores siguientes:</span><span class="sxs-lookup"><span data-stu-id="9b6e6-145">Add a URL type here with the following values:</span></span>
    - <span data-ttu-id="9b6e6-146">**Identificador**: establezca en el valor del identificador del **lote**</span><span class="sxs-lookup"><span data-stu-id="9b6e6-146">**Identifier**: set to the value of your **Bundle identifier**</span></span>
    - <span data-ttu-id="9b6e6-147">**Esquemas de URL**: establezca el URI de redireccionamiento desde el registro de la aplicación que comienza con`msal`</span><span class="sxs-lookup"><span data-stu-id="9b6e6-147">**URL Schemes**: set to the redirect URI from your app registration that begins with `msal`</span></span>
    - <span data-ttu-id="9b6e6-148">**Rol**:`Editor`</span><span class="sxs-lookup"><span data-stu-id="9b6e6-148">**Role**: `Editor`</span></span>
    - <span data-ttu-id="9b6e6-149">**Icono**: dejar vacío</span><span class="sxs-lookup"><span data-stu-id="9b6e6-149">**Icon**: Leave empty</span></span>

![Una captura de pantalla de la sección tipos de URL de info. plist](./images/add-url-type.png)

<span data-ttu-id="9b6e6-151">Por último, actualice el código en el proyecto **GraphTutorial. iOS** para controlar el redireccionamiento durante la autenticación.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-151">Finally, update the code in the **GraphTutorial.iOS** project to handle the redirect during authentication.</span></span> <span data-ttu-id="9b6e6-152">Abra el archivo **AppDelegate.CS** y agregue la siguiente `using` instrucción en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-152">Open the **AppDelegate.cs** file and add the following `using` statement at the top of the file.</span></span>

```cs
using Microsoft.Identity.Client;
```

<span data-ttu-id="9b6e6-153">Agregue la línea siguiente para `FinishedLaunching` que funcione justo antes `return` de la instrucción.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-153">Add the following line to `FinishedLaunching` function just before the `return` statement.</span></span>

```cs
// Specify the Keychain access group
App.PCA.iOSKeychainSecurityGroup = "com.graphdevx.GraphTutorial";
```

<span data-ttu-id="9b6e6-154">Por último, reemplace `OpenUrl` la función para pasar la dirección URL a la biblioteca de MSAL.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-154">Finally, override the `OpenUrl` function to pass the URL to the MSAL library.</span></span> <span data-ttu-id="9b6e6-155">Agregue lo siguiente a la `AppDelegate` clase.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-155">Add the following to the `AppDelegate` class.</span></span>

```cs
// Handling redirect URL
// See: https://github.com/azuread/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics
public override bool OpenUrl(UIApplication app, NSUrl url, NSDictionary options)
{
    AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(url);
    return true;
}
```

## <a name="storing-the-tokens"></a><span data-ttu-id="9b6e6-156">Almacenamiento de tokens</span><span class="sxs-lookup"><span data-stu-id="9b6e6-156">Storing the tokens</span></span>

<span data-ttu-id="9b6e6-157">Cuando se usa la biblioteca de autenticación de Microsoft en un proyecto Xamarin, se aprovecha de forma predeterminada el almacenamiento seguro nativo para almacenar en caché los tokens.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-157">When the Microsoft Authentication Library is used in a Xamarin project, it takes advantage of the native secure storage to cache tokens by default.</span></span> <span data-ttu-id="9b6e6-158">Para obtener más información, vea Serialización de la [caché](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) de tokens.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-158">See [Token cache serialization](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) for more information.</span></span>

## <a name="test-sign-in"></a><span data-ttu-id="9b6e6-159">Inicio de sesión de prueba</span><span class="sxs-lookup"><span data-stu-id="9b6e6-159">Test sign-in</span></span>

<span data-ttu-id="9b6e6-160">En este punto, si ejecuta la aplicación y pulsa el botón **iniciar sesión** , se le pedirá que inicie sesión.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-160">At this point if you run the application and tap the **Sign in** button, you are prompted to sign in.</span></span> <span data-ttu-id="9b6e6-161">Si el inicio de sesión se realiza correctamente, debería ver el token de acceso impreso en el resultado del depurador.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-161">On successful sign in, you should see the access token printed into the debugger's output.</span></span>

![Captura de pantalla de la ventana de resultados en Visual Studio](./images/debugger-access-token.png)

## <a name="get-user-details"></a><span data-ttu-id="9b6e6-163">Obtener detalles del usuario</span><span class="sxs-lookup"><span data-stu-id="9b6e6-163">Get user details</span></span>

<span data-ttu-id="9b6e6-164">Ahora, actualice `SignIn` la función en **app.Xaml.CS** para inicializar el `GraphServiceClient`.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-164">Now update the `SignIn` function in **App.xaml.cs** to initialize the `GraphServiceClient`.</span></span> <span data-ttu-id="9b6e6-165">Agregue el siguiente código antes de `await GetUserInfo();` la línea.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-165">Add the following code before the `await GetUserInfo();` line.</span></span>

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

<span data-ttu-id="9b6e6-166">Ahora, actualice `GetUserInfo` la función para obtener los detalles del usuario de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-166">Now update the `GetUserInfo` function to get the user's details from the Microsoft Graph.</span></span> <span data-ttu-id="9b6e6-167">Reemplace la función `GetUserInfo` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-167">Replace the existing `GetUserInfo` function with the following.</span></span>

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

<span data-ttu-id="9b6e6-168">Si guarda los cambios y ejecuta la aplicación ahora, después de iniciar sesión, la interfaz de usuario se actualizará con el nombre para mostrar y la dirección de correo electrónico del usuario.</span><span class="sxs-lookup"><span data-stu-id="9b6e6-168">If you save your changes and run the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>

