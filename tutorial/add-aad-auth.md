<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="ebcdf-101">En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="ebcdf-102">Esto es necesario para obtener el token de acceso de OAuth necesario para llamar a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="ebcdf-103">En este paso, integrará la [biblioteca de autenticación de Microsoft para .net (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-103">In this step you will integrate the [Microsoft Authentication Library for .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) into the application.</span></span>

<span data-ttu-id="ebcdf-104">En el **Explorador de soluciones**, expanda el proyecto **GraphTutorial** y haga clic con el botón secundario en la carpeta **Models** .</span><span class="sxs-lookup"><span data-stu-id="ebcdf-104">In **Solution Explorer**, expand the **GraphTutorial** project and right-click the **Models** folder.</span></span> <span data-ttu-id="ebcdf-105">Elija **Agregar clase >...** Asigne un nombre `OAuthSettings` a la clase y elija **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-105">Choose **Add > Class...**. Name the class `OAuthSettings` and choose **Add**.</span></span> <span data-ttu-id="ebcdf-106">Abra el archivo **OAuthSettings.CS** y reemplace el contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-106">Open the **OAuthSettings.cs** file and replace its contents with the following.</span></span>

```cs
namespace GraphTutorial.Models
{
    public static class OAuthSettings
    {
        public const string ApplicationId = "YOUR_APP_ID_HERE";
        public const string Scopes = "User.Read Calendars.Read";
    }
}
```

<span data-ttu-id="ebcdf-107">Reemplace `YOUR_APP_ID_HERE` por el identificador de la aplicación del registro de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-107">Replace `YOUR_APP_ID_HERE` with the application ID from your app registration.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="ebcdf-108">Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el `OAuthSettings.cs` archivo del control de código fuente para evitar la pérdida inadvertida del identificador de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-108">If you're using source control such as git, now would be a good time to exclude the `OAuthSettings.cs` file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="ebcdf-109">Implementar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="ebcdf-109">Implement sign-in</span></span>

<span data-ttu-id="ebcdf-110">Abra el archivo **app.Xaml.CS** en el proyecto **GraphTutorial** y agregue las siguientes `using` instrucciones en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-110">Open the **App.xaml.cs** file in the **GraphTutorial** project, and add the following `using` statements to the top of the file.</span></span>

```cs
using GraphTutorial.Models;
using Microsoft.Identity.Client;
using Microsoft.Graph;
using System.Diagnostics;
using System.Linq;
using System.Net.Http.Headers;
```

<span data-ttu-id="ebcdf-111">Agregue las siguientes propiedades a la `App` clase.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-111">Add the following properties to the `App` class.</span></span>

```cs
// UIParent used by Android version of the app
public static object AuthUIParent = null;

// Keychain security group used by iOS version of the app
public static string iOSKeychainSecurityGroup = null;

// Microsoft Authentication client for native/mobile apps
public static IPublicClientApplication PCA;

// Microsoft Graph client
public static GraphServiceClient GraphClient;
```

<span data-ttu-id="ebcdf-112">A continuación, cree un `PublicClientApplication` nuevo en el constructor de `App` la clase.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-112">Next, create a new `PublicClientApplication` in the constructor of the `App` class.</span></span>

```cs
public App()
{
    InitializeComponent();

    var builder = PublicClientApplicationBuilder
        .Create(OAuthSettings.ApplicationId);

    if (!string.IsNullOrEmpty(iOSKeychainSecurityGroup))
    {
        builder = builder.WithIosKeychainSecurityGroup(iOSKeychainSecurityGroup);
    }

    PCA = builder.Build();

    MainPage = new MainPage();
}
```

<span data-ttu-id="ebcdf-113">Ahora, actualice `SignIn` la función para que `PublicClientApplication` use el para obtener un token de acceso.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-113">Now update the `SignIn` function to use the `PublicClientApplication` to get an access token.</span></span> <span data-ttu-id="ebcdf-114">Agregue el siguiente código encima de `await GetUserInfo();` la línea.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-114">Add the following code above the `await GetUserInfo();` line.</span></span>

```cs
var scopes = OAuthSettings.Scopes.Split(' ');

// First, attempt silent sign in
// If the user's information is already in the app's cache,
// they won't have to sign in again.
string accessToken = string.Empty;
try
{
    var accounts = await PCA.GetAccountsAsync();
    if (accounts.Count() > 0)
    {
        var silentAuthResult = await PCA
            .AcquireTokenSilent(scopes, accounts.FirstOrDefault())
            .ExecuteAsync();

        Debug.WriteLine("User already signed in.");
        Debug.WriteLine($"Access token: {silentAuthResult.AccessToken}");
        accessToken = silentAuthResult.AccessToken;
    }
}
catch (MsalUiRequiredException)
{
    // This exception is thrown when an interactive sign-in is required.
    Debug.WriteLine("Silent token request failed, user needs to sign-in");
}

if (string.IsNullOrEmpty(accessToken))
{
    // Prompt the user to sign-in
    var interactiveRequest = PCA.AcquireTokenInteractive(scopes);

    if (AuthUIParent != null)
    {
        interactiveRequest = interactiveRequest
            .WithParentActivityOrWindow(AuthUIParent);
    }

    var authResult = await interactiveRequest.ExecuteAsync();
    Debug.WriteLine($"Access Token: {authResult.AccessToken}");
}
```

<span data-ttu-id="ebcdf-115">Este código primero intenta obtener un token de acceso en modo silencioso.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-115">This code first attempts to get an access token silently.</span></span> <span data-ttu-id="ebcdf-116">Si la información de un usuario ya se encuentra en la memoria caché de la aplicación (por ejemplo, si el usuario ha cerrado la aplicación anteriormente sin cerrar sesión), esto se realizará correctamente y no hay motivo para preguntar al usuario.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-116">If a user's information is already in the app's cache (for example, if the user closed the app previously without signing out), this will succeed, and there's no reason to prompt the user.</span></span> <span data-ttu-id="ebcdf-117">Si no hay información de un usuario en la memoria caché, la `AcquireTokenSilent().ExecuteAsync()` función inicia una `MsalUiRequiredException`.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-117">If there is not a user's information in the cache, the `AcquireTokenSilent().ExecuteAsync()` function throws an `MsalUiRequiredException`.</span></span> <span data-ttu-id="ebcdf-118">En este caso, el código llama a la función interactiva para obtener un `AcquireTokenInteractive`token,.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-118">In this case, the code calls the interactive function to get a token, `AcquireTokenInteractive`.</span></span>

<span data-ttu-id="ebcdf-119">Ahora, actualice `SignOut` la función para quitar la información del usuario de la memoria caché.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-119">Now update the `SignOut` function to remove the user's information from the cache.</span></span> <span data-ttu-id="ebcdf-120">Agregue el código siguiente al principio de la `SignOut` función.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-120">Add the following code to the beginning of the `SignOut` function.</span></span>

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

### <a name="update-android-project-to-enable-sign-in"></a><span data-ttu-id="ebcdf-121">Actualizar el proyecto de Android para habilitar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="ebcdf-121">Update Android project to enable sign-in</span></span>

<span data-ttu-id="ebcdf-122">Cuando se usa en un proyecto de Xamarin Android, la biblioteca de autenticación de Microsoft tiene algunos [requisitos específicos de Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).</span><span class="sxs-lookup"><span data-stu-id="ebcdf-122">When used in a Xamarin Android project, the Microsoft Authentication Library has a few [requirements specific to Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).</span></span>

<span data-ttu-id="ebcdf-123">En primer lugar, debe actualizar el manifiesto de Android para registrar el URI de redireccionamiento.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-123">First, you need to update the Android manifest to register the redirect URI.</span></span> <span data-ttu-id="ebcdf-124">En el proyecto de **GraphTutorial. Android** , expanda la carpeta **propiedades** y Abra **AndroidManifest. XML**.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-124">In **GraphTutorial.Android** project, expand the **Properties** folder, then open **AndroidManifest.xml**.</span></span> <span data-ttu-id="ebcdf-125">Si está usando Visual Studio para Mac, cambie a la vista de **origen** mediante las pestañas de la parte inferior del archivo.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-125">If you're using Visual Studio for Mac, switch to the **Source** view using the tabs at the bottom of the file.</span></span> <span data-ttu-id="ebcdf-126">Reemplace todo el contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-126">Replace the entire contents with the following.</span></span>

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" android:versionCode="1" android:versionName="1.0" package="com.companyname.GraphTutorial">
    <uses-sdk android:minSdkVersion="21" android:targetSdkVersion="28" />
    <application android:label="GraphTutorial.Android">
        <activity android:name="microsoft.identity.client.BrowserTabActivity">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data android:scheme="msalYOUR_APP_ID_HERE" android:host="auth" />
            </intent-filter>
        </activity>
    </application>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.INTERNET" />
</manifest>
```

<span data-ttu-id="ebcdf-127">Reemplace `YOUR_APP_ID_HERE` con el identificador de la aplicación del registro de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-127">Replace `YOUR_APP_ID_HERE` with your application ID from your app registration.</span></span>

<span data-ttu-id="ebcdf-128">A continuación, Abra **MainActivity.CS** y agregue las `using` siguientes instrucciones en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-128">Next, open **MainActivity.cs** and add the following `using` statements to the top of the file.</span></span>

```cs
using Microsoft.Identity.Client;
using Android.Content;
```

<span data-ttu-id="ebcdf-129">A continuación, reemplace `OnActivityResult` la función para pasar el control a la biblioteca de MSAL.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-129">Then, override the `OnActivityResult` function to pass control to the MSAL library.</span></span> <span data-ttu-id="ebcdf-130">Agregue lo siguiente a la `MainActivity` clase.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-130">Add the following to the `MainActivity` class.</span></span>

```cs
protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
{
    base.OnActivityResult(requestCode, resultCode, data);
    AuthenticationContinuationHelper
        .SetAuthenticationContinuationEventArgs(requestCode, resultCode, data);
}
```

<span data-ttu-id="ebcdf-131">Por último, en `OnCreate` la función, agregue la siguiente línea después `LoadApplication(new App());` de la línea.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-131">Finally, in the `OnCreate` function, add the following line after the `LoadApplication(new App());` line.</span></span>

```cs
App.AuthUIParent = this;
```

### <a name="update-ios-project-to-enable-sign-in"></a><span data-ttu-id="ebcdf-132">Actualizar el proyecto de iOS para habilitar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="ebcdf-132">Update iOS project to enable sign-in</span></span>

> [!IMPORTANT]
> <span data-ttu-id="ebcdf-133">Como MSAL requiere el uso de un archivo de derechos. plist, debe configurar Visual Studio con su cuenta de desarrollador de Apple para habilitar el aprovisionamiento.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-133">Because MSAL requires use of an Entitlements.plist file, you must configure Visual Studio with your Apple developer account to enable provisioning.</span></span> <span data-ttu-id="ebcdf-134">Si ejecuta este tutorial en el simulador de iPhone, debe agregar **derechos. plist** en el campo **derechos personalizados** en la configuración del proyecto de **GraphTutorial. iOS** , la firma del lote de **compilación >iOS**.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-134">If you are running this tutorial in the iPhone simulator, you need to add **Entitlements.plist** in the **Custom Entitlements** field in **GraphTutorial.iOS** project's settings, **Build->iOS Bundle Signing**.</span></span> <span data-ttu-id="ebcdf-135">Para obtener más información, consulte [aprovisionamiento de dispositivos para Xamarin. iOS](/xamarin/ios/get-started/installation/device-provisioning).</span><span class="sxs-lookup"><span data-stu-id="ebcdf-135">For more information, see [Device provisioning for Xamarin.iOS](/xamarin/ios/get-started/installation/device-provisioning).</span></span>

<span data-ttu-id="ebcdf-136">Cuando se usa en un proyecto de Xamarin iOS, la biblioteca de autenticación de Microsoft tiene algunos [requisitos específicos de iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).</span><span class="sxs-lookup"><span data-stu-id="ebcdf-136">When used in a Xamarin iOS project, the Microsoft Authentication Library has a few [requirements specific to iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).</span></span>

<span data-ttu-id="ebcdf-137">En primer lugar, debe habilitar el acceso a llaves.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-137">First, you need to enable Keychain access.</span></span> <span data-ttu-id="ebcdf-138">En el explorador de soluciones, expanda el proyecto **GraphTutorial. iOS** y, a continuación, abra el archivo contitles **. plist** .</span><span class="sxs-lookup"><span data-stu-id="ebcdf-138">In Solution Explorer, expand the **GraphTutorial.iOS** project, then open the **Entitlements.plist** file.</span></span> <span data-ttu-id="ebcdf-139">Busque el derecho de la **cadena de claves** y seleccione **Habilitar cadena de claves**.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-139">Locate the **Keychain** entitlement, and select **Enable KeyChain**.</span></span> <span data-ttu-id="ebcdf-140">En **grupos de llaves**, agregue una entrada con el `com.YOUR_DOMAIN.GraphTutorial`formato.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-140">In **Keychain Groups**, add an entry with the format `com.YOUR_DOMAIN.GraphTutorial`.</span></span>

![Captura de pantalla de la configuración de derechos de llaves](./images/enable-keychain-access.png)

<span data-ttu-id="ebcdf-142">A continuación, debe registrar el URI de redireccionamiento de MSAL predeterminado como un tipo de dirección URL que la aplicación controla.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-142">Next, you need to register the default MSAL redirect URI as a URL type that your app handles.</span></span> <span data-ttu-id="ebcdf-143">Abra el archivo **info. plist** y realice los siguientes cambios.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-143">Open the **Info.plist** file and make the following changes.</span></span>

- <span data-ttu-id="ebcdf-144">En la pestaña **aplicación** , compruebe que el valor del **identificador de lote** coincida con el valor que estableció para los grupos de **llaves** en **derechos. plist**.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-144">On the **Application** tab, check that the value of **Bundle identifier** matches the value you set for **Keychain Groups** in **Entitlements.plist**.</span></span> <span data-ttu-id="ebcdf-145">Si no es así, actualícelo ahora.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-145">If it doesn't, update it now.</span></span>
- <span data-ttu-id="ebcdf-146">En la ficha **avanzadas** , busque la sección **tipos de URL** .</span><span class="sxs-lookup"><span data-stu-id="ebcdf-146">On the **Advanced** tab, locate the **URL Types** section.</span></span> <span data-ttu-id="ebcdf-147">Agregue aquí un tipo de dirección URL con los valores siguientes:</span><span class="sxs-lookup"><span data-stu-id="ebcdf-147">Add a URL type here with the following values:</span></span>
  - <span data-ttu-id="ebcdf-148">**Identificador**: establezca en el valor del identificador del **lote**</span><span class="sxs-lookup"><span data-stu-id="ebcdf-148">**Identifier**: set to the value of your **Bundle identifier**</span></span>
  - <span data-ttu-id="ebcdf-149">**Esquemas de dirección URL**: establezca en `msal{YOUR-APP-ID}`.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-149">**URL Schemes**: set to `msal{YOUR-APP-ID}`.</span></span> <span data-ttu-id="ebcdf-150">Por ejemplo, si el identificador de la `67ad5eba-0cfc-414d-8f9f-0a6d973a907c`aplicación es, establecería en `msal67ad5eba-0cfc-414d-8f9f-0a6d973a907c`.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-150">For example, if your app ID is `67ad5eba-0cfc-414d-8f9f-0a6d973a907c`, you would set this to `msal67ad5eba-0cfc-414d-8f9f-0a6d973a907c`.</span></span>
  - <span data-ttu-id="ebcdf-151">**Rol**:`Editor`</span><span class="sxs-lookup"><span data-stu-id="ebcdf-151">**Role**: `Editor`</span></span>
  - <span data-ttu-id="ebcdf-152">**Icono**: dejar vacío</span><span class="sxs-lookup"><span data-stu-id="ebcdf-152">**Icon**: Leave empty</span></span>

![Una captura de pantalla de la sección tipos de URL de info. plist](./images/add-url-type.png)

<span data-ttu-id="ebcdf-154">Por último, actualice el código en el proyecto **GraphTutorial. iOS** para controlar el redireccionamiento durante la autenticación.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-154">Finally, update the code in the **GraphTutorial.iOS** project to handle the redirect during authentication.</span></span> <span data-ttu-id="ebcdf-155">Abra el archivo **AppDelegate.CS** y agregue la siguiente `using` instrucción en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-155">Open the **AppDelegate.cs** file and add the following `using` statement at the top of the file.</span></span>

```cs
using Microsoft.Identity.Client;
```

<span data-ttu-id="ebcdf-156">Agregue la línea siguiente para `FinishedLaunching` que funcione justo antes `LoadApplication(new App());` de la línea.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-156">Add the following line to `FinishedLaunching` function just before the `LoadApplication(new App());` line.</span></span>

```cs
// Specify the Keychain access group
App.iOSKeychainSecurityGroup = NSBundle.MainBundle.BundleIdentifier;
```

<span data-ttu-id="ebcdf-157">Por último, reemplace `OpenUrl` la función para pasar la dirección URL a la biblioteca de MSAL.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-157">Finally, override the `OpenUrl` function to pass the URL to the MSAL library.</span></span> <span data-ttu-id="ebcdf-158">Agregue lo siguiente a la `AppDelegate` clase.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-158">Add the following to the `AppDelegate` class.</span></span>

```cs
// Handling redirect URL
// See: https://github.com/azuread/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics
public override bool OpenUrl(UIApplication app, NSUrl url, NSDictionary options)
{
    AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(url);
    return true;
}
```

## <a name="storing-the-tokens"></a><span data-ttu-id="ebcdf-159">Almacenamiento de tokens</span><span class="sxs-lookup"><span data-stu-id="ebcdf-159">Storing the tokens</span></span>

<span data-ttu-id="ebcdf-160">Cuando se usa la biblioteca de autenticación de Microsoft en un proyecto Xamarin, se aprovecha de forma predeterminada el almacenamiento seguro nativo para almacenar en caché los tokens.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-160">When the Microsoft Authentication Library is used in a Xamarin project, it takes advantage of the native secure storage to cache tokens by default.</span></span> <span data-ttu-id="ebcdf-161">Para obtener más información, vea Serialización de la [caché](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) de tokens.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-161">See [Token cache serialization](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) for more information.</span></span>

## <a name="test-sign-in"></a><span data-ttu-id="ebcdf-162">Inicio de sesión de prueba</span><span class="sxs-lookup"><span data-stu-id="ebcdf-162">Test sign-in</span></span>

<span data-ttu-id="ebcdf-163">En este punto, si ejecuta la aplicación y pulsa el botón **iniciar sesión** , se le pedirá que inicie sesión.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-163">At this point if you run the application and tap the **Sign in** button, you are prompted to sign in.</span></span> <span data-ttu-id="ebcdf-164">Si el inicio de sesión se realiza correctamente, debería ver el token de acceso impreso en el resultado del depurador.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-164">On successful sign in, you should see the access token printed into the debugger's output.</span></span>

![Captura de pantalla de la ventana de resultados en Visual Studio](./images/debugger-access-token.png)

## <a name="get-user-details"></a><span data-ttu-id="ebcdf-166">Obtener detalles del usuario</span><span class="sxs-lookup"><span data-stu-id="ebcdf-166">Get user details</span></span>

<span data-ttu-id="ebcdf-167">Ahora, actualice `SignIn` la función en **app.Xaml.CS** para inicializar el `GraphServiceClient`.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-167">Now update the `SignIn` function in **App.xaml.cs** to initialize the `GraphServiceClient`.</span></span> <span data-ttu-id="ebcdf-168">Agregue el siguiente código antes de `await GetUserInfo();` la línea.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-168">Add the following code before the `await GetUserInfo();` line.</span></span>

```cs
// Initialize Graph client
GraphClient = new GraphServiceClient(new DelegateAuthenticationProvider(
    async (requestMessage) =>
    {
        var accounts = await PCA.GetAccountsAsync();

        var result = await PCA.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
            .ExecuteAsync();

        requestMessage.Headers.Authorization =
            new AuthenticationHeaderValue("Bearer", result.AccessToken);
    }));
```

<span data-ttu-id="ebcdf-169">Ahora, actualice `GetUserInfo` la función para obtener los detalles del usuario de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-169">Now update the `GetUserInfo` function to get the user's details from the Microsoft Graph.</span></span> <span data-ttu-id="ebcdf-170">Reemplace la función `GetUserInfo` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-170">Replace the existing `GetUserInfo` function with the following.</span></span>

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

<span data-ttu-id="ebcdf-171">Si guarda los cambios y ejecuta la aplicación ahora, después de iniciar sesión, la interfaz de usuario se actualizará con el nombre para mostrar y la dirección de correo electrónico del usuario.</span><span class="sxs-lookup"><span data-stu-id="ebcdf-171">If you save your changes and run the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
