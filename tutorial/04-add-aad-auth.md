<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="7ca30-101">En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="7ca30-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="7ca30-102">Esto es necesario para obtener el token de acceso de OAuth necesario para llamar a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="7ca30-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="7ca30-103">En este paso, integrará la [biblioteca de autenticación de Microsoft para .net (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="7ca30-103">In this step you will integrate the [Microsoft Authentication Library for .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) into the application.</span></span>

<span data-ttu-id="7ca30-104">En el **Explorador de soluciones**, expanda el proyecto **GraphTutorial** y haga clic con el botón secundario en la carpeta **Models** .</span><span class="sxs-lookup"><span data-stu-id="7ca30-104">In **Solution Explorer**, expand the **GraphTutorial** project and right-click the **Models** folder.</span></span> <span data-ttu-id="7ca30-105">Seleccione **agregar > clase...**. Asigne un nombre `OAuthSettings` a la clase y seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="7ca30-105">Select **Add > Class...**. Name the class `OAuthSettings` and select **Add**.</span></span> <span data-ttu-id="7ca30-106">Abra el archivo **OAuthSettings.CS** y reemplace el contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="7ca30-106">Open the **OAuthSettings.cs** file and replace its contents with the following.</span></span>

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

<span data-ttu-id="7ca30-107">Reemplace `YOUR_APP_ID_HERE` por el identificador de la aplicación del registro de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="7ca30-107">Replace `YOUR_APP_ID_HERE` with the application ID from your app registration.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="7ca30-108">Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el `OAuthSettings.cs` archivo del control de código fuente para evitar la pérdida inadvertida del identificador de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="7ca30-108">If you're using source control such as git, now would be a good time to exclude the `OAuthSettings.cs` file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="7ca30-109">Implementar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="7ca30-109">Implement sign-in</span></span>

<span data-ttu-id="7ca30-110">Abra el archivo **app.Xaml.CS** en el proyecto **GraphTutorial** y agregue las siguientes `using` instrucciones en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="7ca30-110">Open the **App.xaml.cs** file in the **GraphTutorial** project, and add the following `using` statements to the top of the file.</span></span>

```cs
using GraphTutorial.Models;
using Microsoft.Identity.Client;
using Microsoft.Graph;
using System.Diagnostics;
using System.Linq;
using System.Net.Http.Headers;
```

<span data-ttu-id="7ca30-111">Cambie la línea de declaración de clase de **aplicación** para resolver el conflicto de nombres de la **aplicación**.</span><span class="sxs-lookup"><span data-stu-id="7ca30-111">Change the **App** class declaration line to resolve the name conflict for **Application**.</span></span>

```cs
public partial class App : Xamarin.Forms.Application, INotifyPropertyChanged
```

<span data-ttu-id="7ca30-112">Agregue las siguientes propiedades a la `App` clase.</span><span class="sxs-lookup"><span data-stu-id="7ca30-112">Add the following properties to the `App` class.</span></span>

```cs
// UIParent used by Android version of the app
public static object AuthUIParent = null;

// Keychain security group used by iOS version of the app
public static string iOSKeychainSecurityGroup = null;

// Microsoft Authentication client for native/mobile apps
public static IPublicClientApplication PCA;

// Microsoft Graph client
public static GraphServiceClient GraphClient;

// Microsoft Graph permissions used by app
private readonly string[] Scopes = OAuthSettings.Scopes.Split(' ');
```

<span data-ttu-id="7ca30-113">A continuación, cree un `PublicClientApplication` nuevo en el constructor de `App` la clase.</span><span class="sxs-lookup"><span data-stu-id="7ca30-113">Next, create a new `PublicClientApplication` in the constructor of the `App` class.</span></span>

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

<span data-ttu-id="7ca30-114">Ahora, actualice `SignIn` la función para que `PublicClientApplication` use el para obtener un token de acceso.</span><span class="sxs-lookup"><span data-stu-id="7ca30-114">Now update the `SignIn` function to use the `PublicClientApplication` to get an access token.</span></span> <span data-ttu-id="7ca30-115">Agregue el siguiente código encima de `await GetUserInfo();` la línea.</span><span class="sxs-lookup"><span data-stu-id="7ca30-115">Add the following code above the `await GetUserInfo();` line.</span></span>

```cs
// First, attempt silent sign in
// If the user's information is already in the app's cache,
// they won't have to sign in again.
try
{
    var accounts = await PCA.GetAccountsAsync();

    var silentAuthResult = await PCA
        .AcquireTokenSilent(Scopes, accounts.FirstOrDefault())
        .ExecuteAsync();

    Debug.WriteLine("User already signed in.");
    Debug.WriteLine($"Successful silent authentication for: {silentAuthResult.Account.Username}");
    Debug.WriteLine($"Access token: {silentAuthResult.AccessToken}");
}
catch (MsalUiRequiredException msalEx)
{
    // This exception is thrown when an interactive sign-in is required.
    Debug.WriteLine("Silent token request failed, user needs to sign-in: " + msalEx.Message);
    // Prompt the user to sign-in
    var interactiveRequest = PCA.AcquireTokenInteractive(Scopes);

    if (AuthUIParent != null)
    {
        interactiveRequest = interactiveRequest
            .WithParentActivityOrWindow(AuthUIParent);
    }

    var interactiveAuthResult = await interactiveRequest.ExecuteAsync();
    Debug.WriteLine($"Successful interactive authentication for: {interactiveAuthResult.Account.Username}");
    Debug.WriteLine($"Access token: {interactiveAuthResult.AccessToken}");
}
catch (Exception ex)
{
    Debug.WriteLine("Authentication failed. See exception messsage for more details: " + ex.Message);
}
```

<span data-ttu-id="7ca30-116">Este código primero intenta obtener un token de acceso en modo silencioso.</span><span class="sxs-lookup"><span data-stu-id="7ca30-116">This code first attempts to get an access token silently.</span></span> <span data-ttu-id="7ca30-117">Si la información de un usuario ya se encuentra en la memoria caché de la aplicación (por ejemplo, si el usuario ha cerrado la aplicación anteriormente sin cerrar sesión), esto se realizará correctamente y no hay motivo para preguntar al usuario.</span><span class="sxs-lookup"><span data-stu-id="7ca30-117">If a user's information is already in the app's cache (for example, if the user closed the app previously without signing out), this will succeed, and there's no reason to prompt the user.</span></span> <span data-ttu-id="7ca30-118">Si no hay información de un usuario en la memoria caché, la `AcquireTokenSilent().ExecuteAsync()` función inicia una `MsalUiRequiredException`.</span><span class="sxs-lookup"><span data-stu-id="7ca30-118">If there is not a user's information in the cache, the `AcquireTokenSilent().ExecuteAsync()` function throws an `MsalUiRequiredException`.</span></span> <span data-ttu-id="7ca30-119">En este caso, el código llama a la función interactiva para obtener un `AcquireTokenInteractive`token,.</span><span class="sxs-lookup"><span data-stu-id="7ca30-119">In this case, the code calls the interactive function to get a token, `AcquireTokenInteractive`.</span></span>

<span data-ttu-id="7ca30-120">Ahora, actualice `SignOut` la función para quitar la información del usuario de la memoria caché.</span><span class="sxs-lookup"><span data-stu-id="7ca30-120">Now update the `SignOut` function to remove the user's information from the cache.</span></span> <span data-ttu-id="7ca30-121">Agregue el código siguiente al principio de la `SignOut` función.</span><span class="sxs-lookup"><span data-stu-id="7ca30-121">Add the following code to the beginning of the `SignOut` function.</span></span>

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

### <a name="update-android-project-to-enable-sign-in"></a><span data-ttu-id="7ca30-122">Actualizar el proyecto de Android para habilitar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="7ca30-122">Update Android project to enable sign-in</span></span>

<span data-ttu-id="7ca30-123">Cuando se usa en un proyecto de Xamarin Android, la biblioteca de autenticación de Microsoft tiene algunos [requisitos específicos de Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).</span><span class="sxs-lookup"><span data-stu-id="7ca30-123">When used in a Xamarin Android project, the Microsoft Authentication Library has a few [requirements specific to Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).</span></span>

<span data-ttu-id="7ca30-124">En primer lugar, debe actualizar el manifiesto de Android para registrar el URI de redireccionamiento.</span><span class="sxs-lookup"><span data-stu-id="7ca30-124">First, you need to update the Android manifest to register the redirect URI.</span></span> <span data-ttu-id="7ca30-125">En el proyecto de **GraphTutorial. Android** , expanda la carpeta **propiedades** y Abra **AndroidManifest. XML**.</span><span class="sxs-lookup"><span data-stu-id="7ca30-125">In **GraphTutorial.Android** project, expand the **Properties** folder, then open **AndroidManifest.xml**.</span></span> <span data-ttu-id="7ca30-126">Si está usando Visual Studio para Mac, cambie a la vista de **origen** mediante las pestañas de la parte inferior del archivo.</span><span class="sxs-lookup"><span data-stu-id="7ca30-126">If you're using Visual Studio for Mac, switch to the **Source** view using the tabs at the bottom of the file.</span></span> <span data-ttu-id="7ca30-127">Reemplace todo el contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="7ca30-127">Replace the entire contents with the following.</span></span>

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

<span data-ttu-id="7ca30-128">Reemplace `YOUR_APP_ID_HERE` con el identificador de la aplicación del registro de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="7ca30-128">Replace `YOUR_APP_ID_HERE` with your application ID from your app registration.</span></span>

<span data-ttu-id="7ca30-129">A continuación, Abra **MainActivity.CS** y agregue las `using` siguientes instrucciones en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="7ca30-129">Next, open **MainActivity.cs** and add the following `using` statements to the top of the file.</span></span>

```cs
using Microsoft.Identity.Client;
using Android.Content;
```

<span data-ttu-id="7ca30-130">A continuación, reemplace `OnActivityResult` la función para pasar el control a la biblioteca de MSAL.</span><span class="sxs-lookup"><span data-stu-id="7ca30-130">Then, override the `OnActivityResult` function to pass control to the MSAL library.</span></span> <span data-ttu-id="7ca30-131">Agregue lo siguiente a la `MainActivity` clase.</span><span class="sxs-lookup"><span data-stu-id="7ca30-131">Add the following to the `MainActivity` class.</span></span>

```cs
protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
{
    base.OnActivityResult(requestCode, resultCode, data);
    AuthenticationContinuationHelper
        .SetAuthenticationContinuationEventArgs(requestCode, resultCode, data);
}
```

<span data-ttu-id="7ca30-132">Por último, en `OnCreate` la función, agregue la siguiente línea después `LoadApplication(new App());` de la línea.</span><span class="sxs-lookup"><span data-stu-id="7ca30-132">Finally, in the `OnCreate` function, add the following line after the `LoadApplication(new App());` line.</span></span>

```cs
App.AuthUIParent = this;
```

### <a name="update-ios-project-to-enable-sign-in"></a><span data-ttu-id="7ca30-133">Actualizar el proyecto de iOS para habilitar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="7ca30-133">Update iOS project to enable sign-in</span></span>

> [!IMPORTANT]
> <span data-ttu-id="7ca30-134">Como MSAL requiere el uso de un archivo de derechos. plist, debe configurar Visual Studio con su cuenta de desarrollador de Apple para habilitar el aprovisionamiento.</span><span class="sxs-lookup"><span data-stu-id="7ca30-134">Because MSAL requires use of an Entitlements.plist file, you must configure Visual Studio with your Apple developer account to enable provisioning.</span></span> <span data-ttu-id="7ca30-135">Si está ejecutando este tutorial en el simulador de iPhone, debe agregar **derechos. plist** en el campo **derechos personalizados** en la configuración del proyecto de **GraphTutorial. iOS** , **compilación->firma del paquete de iOS**.</span><span class="sxs-lookup"><span data-stu-id="7ca30-135">If you are running this tutorial in the iPhone simulator, you need to add **Entitlements.plist** in the **Custom Entitlements** field in **GraphTutorial.iOS** project's settings, **Build->iOS Bundle Signing**.</span></span> <span data-ttu-id="7ca30-136">Para obtener más información, consulte [aprovisionamiento de dispositivos para Xamarin. iOS](/xamarin/ios/get-started/installation/device-provisioning).</span><span class="sxs-lookup"><span data-stu-id="7ca30-136">For more information, see [Device provisioning for Xamarin.iOS](/xamarin/ios/get-started/installation/device-provisioning).</span></span>

<span data-ttu-id="7ca30-137">Cuando se usa en un proyecto de Xamarin iOS, la biblioteca de autenticación de Microsoft tiene algunos [requisitos específicos de iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).</span><span class="sxs-lookup"><span data-stu-id="7ca30-137">When used in a Xamarin iOS project, the Microsoft Authentication Library has a few [requirements specific to iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).</span></span>

<span data-ttu-id="7ca30-138">En primer lugar, debe habilitar el acceso a llaves.</span><span class="sxs-lookup"><span data-stu-id="7ca30-138">First, you need to enable Keychain access.</span></span> <span data-ttu-id="7ca30-139">En el explorador de soluciones, expanda el proyecto **GraphTutorial. iOS** y, a continuación, abra el archivo **contitles. plist** .</span><span class="sxs-lookup"><span data-stu-id="7ca30-139">In Solution Explorer, expand the **GraphTutorial.iOS** project, then open the **Entitlements.plist** file.</span></span> <span data-ttu-id="7ca30-140">Busque el derecho de la **cadena de claves** y seleccione **Habilitar cadena de claves**.</span><span class="sxs-lookup"><span data-stu-id="7ca30-140">Locate the **Keychain** entitlement, and select **Enable KeyChain**.</span></span> <span data-ttu-id="7ca30-141">En **grupos de llaves**, agregue una entrada con el `com.YOUR_DOMAIN.GraphTutorial`formato.</span><span class="sxs-lookup"><span data-stu-id="7ca30-141">In **Keychain Groups**, add an entry with the format `com.YOUR_DOMAIN.GraphTutorial`.</span></span>

![Captura de pantalla de la configuración de derechos de llaves](./images/enable-keychain-access.png)

<span data-ttu-id="7ca30-143">A continuación, debe registrar el URI de redireccionamiento de MSAL predeterminado como un tipo de dirección URL que la aplicación controla.</span><span class="sxs-lookup"><span data-stu-id="7ca30-143">Next, you need to register the default MSAL redirect URI as a URL type that your app handles.</span></span> <span data-ttu-id="7ca30-144">Abra el archivo **info. plist** y realice los siguientes cambios.</span><span class="sxs-lookup"><span data-stu-id="7ca30-144">Open the **Info.plist** file and make the following changes.</span></span>

- <span data-ttu-id="7ca30-145">En la pestaña **aplicación** , compruebe que el valor del **identificador de lote** coincida con el valor que estableció para los grupos de **llaves** en **derechos. plist**.</span><span class="sxs-lookup"><span data-stu-id="7ca30-145">On the **Application** tab, check that the value of **Bundle identifier** matches the value you set for **Keychain Groups** in **Entitlements.plist**.</span></span> <span data-ttu-id="7ca30-146">Si no es así, actualícelo ahora.</span><span class="sxs-lookup"><span data-stu-id="7ca30-146">If it doesn't, update it now.</span></span>
- <span data-ttu-id="7ca30-147">En la ficha **avanzadas** , busque la sección **tipos de URL** .</span><span class="sxs-lookup"><span data-stu-id="7ca30-147">On the **Advanced** tab, locate the **URL Types** section.</span></span> <span data-ttu-id="7ca30-148">Agregue aquí un tipo de dirección URL con los valores siguientes:</span><span class="sxs-lookup"><span data-stu-id="7ca30-148">Add a URL type here with the following values:</span></span>
  - <span data-ttu-id="7ca30-149">**Identificador**: establezca en el valor del identificador del **lote**</span><span class="sxs-lookup"><span data-stu-id="7ca30-149">**Identifier**: set to the value of your **Bundle identifier**</span></span>
  - <span data-ttu-id="7ca30-150">**Esquemas de dirección URL**: establezca en `msal{YOUR-APP-ID}`.</span><span class="sxs-lookup"><span data-stu-id="7ca30-150">**URL Schemes**: set to `msal{YOUR-APP-ID}`.</span></span> <span data-ttu-id="7ca30-151">Por ejemplo, si el identificador de la `67ad5eba-0cfc-414d-8f9f-0a6d973a907c`aplicación es, establecería en `msal67ad5eba-0cfc-414d-8f9f-0a6d973a907c`.</span><span class="sxs-lookup"><span data-stu-id="7ca30-151">For example, if your app ID is `67ad5eba-0cfc-414d-8f9f-0a6d973a907c`, you would set this to `msal67ad5eba-0cfc-414d-8f9f-0a6d973a907c`.</span></span>
  - <span data-ttu-id="7ca30-152">**Rol**:`Editor`</span><span class="sxs-lookup"><span data-stu-id="7ca30-152">**Role**: `Editor`</span></span>
  - <span data-ttu-id="7ca30-153">**Icono**: dejar vacío</span><span class="sxs-lookup"><span data-stu-id="7ca30-153">**Icon**: Leave empty</span></span>

![Una captura de pantalla de la sección tipos de URL de info. plist](./images/add-url-type.png)

<span data-ttu-id="7ca30-155">Por último, actualice el código en el proyecto **GraphTutorial. iOS** para controlar el redireccionamiento durante la autenticación.</span><span class="sxs-lookup"><span data-stu-id="7ca30-155">Finally, update the code in the **GraphTutorial.iOS** project to handle the redirect during authentication.</span></span> <span data-ttu-id="7ca30-156">Abra el archivo **AppDelegate.CS** y agregue la siguiente `using` instrucción en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="7ca30-156">Open the **AppDelegate.cs** file and add the following `using` statement at the top of the file.</span></span>

```cs
using Microsoft.Identity.Client;
```

<span data-ttu-id="7ca30-157">Agregue la línea siguiente para `FinishedLaunching` que funcione justo antes `LoadApplication(new App());` de la línea.</span><span class="sxs-lookup"><span data-stu-id="7ca30-157">Add the following line to `FinishedLaunching` function just before the `LoadApplication(new App());` line.</span></span>

```cs
// Specify the Keychain access group
App.iOSKeychainSecurityGroup = NSBundle.MainBundle.BundleIdentifier;
```

<span data-ttu-id="7ca30-158">Por último, reemplace `OpenUrl` la función para pasar la dirección URL a la biblioteca de MSAL.</span><span class="sxs-lookup"><span data-stu-id="7ca30-158">Finally, override the `OpenUrl` function to pass the URL to the MSAL library.</span></span> <span data-ttu-id="7ca30-159">Agregue lo siguiente a la `AppDelegate` clase.</span><span class="sxs-lookup"><span data-stu-id="7ca30-159">Add the following to the `AppDelegate` class.</span></span>

```cs
// Handling redirect URL
// See: https://github.com/azuread/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics
public override bool OpenUrl(UIApplication app, NSUrl url, NSDictionary options)
{
    AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(url);
    return true;
}
```

## <a name="storing-the-tokens"></a><span data-ttu-id="7ca30-160">Almacenamiento de tokens</span><span class="sxs-lookup"><span data-stu-id="7ca30-160">Storing the tokens</span></span>

<span data-ttu-id="7ca30-161">Cuando se usa la biblioteca de autenticación de Microsoft en un proyecto Xamarin, se aprovecha de forma predeterminada el almacenamiento seguro nativo para almacenar en caché los tokens.</span><span class="sxs-lookup"><span data-stu-id="7ca30-161">When the Microsoft Authentication Library is used in a Xamarin project, it takes advantage of the native secure storage to cache tokens by default.</span></span> <span data-ttu-id="7ca30-162">Para obtener más información, vea [serialización de la caché de tokens](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) .</span><span class="sxs-lookup"><span data-stu-id="7ca30-162">See [Token cache serialization](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) for more information.</span></span>

## <a name="test-sign-in"></a><span data-ttu-id="7ca30-163">Inicio de sesión de prueba</span><span class="sxs-lookup"><span data-stu-id="7ca30-163">Test sign-in</span></span>

<span data-ttu-id="7ca30-164">En este punto, si ejecuta la aplicación y pulsa el botón **iniciar sesión** , se le pedirá que inicie sesión.</span><span class="sxs-lookup"><span data-stu-id="7ca30-164">At this point if you run the application and tap the **Sign in** button, you are prompted to sign in.</span></span> <span data-ttu-id="7ca30-165">Si el inicio de sesión se realiza correctamente, debería ver el token de acceso impreso en el resultado del depurador.</span><span class="sxs-lookup"><span data-stu-id="7ca30-165">On successful sign in, you should see the access token printed into the debugger's output.</span></span>

![Captura de pantalla de la ventana de resultados en Visual Studio](./images/debugger-access-token.png)

## <a name="get-user-details"></a><span data-ttu-id="7ca30-167">Obtener detalles del usuario</span><span class="sxs-lookup"><span data-stu-id="7ca30-167">Get user details</span></span>

<span data-ttu-id="7ca30-168">Agregue una nueva función a la clase **App** para inicializar `GraphServiceClient`.</span><span class="sxs-lookup"><span data-stu-id="7ca30-168">Add a new function to the **App** class to initialize the `GraphServiceClient`.</span></span>

```cs
private async Task InitializeGraphClientAsync()
{
    var currentAccounts = await PCA.GetAccountsAsync();
    try
    {
        if (currentAccounts.Count() > 0)
        {
            // Initialize Graph client
            GraphClient = new GraphServiceClient(new DelegateAuthenticationProvider(
                async (requestMessage) =>
                {
                    var result = await PCA.AcquireTokenSilent(Scopes, currentAccounts.FirstOrDefault())
                        .ExecuteAsync();

                    requestMessage.Headers.Authorization =
                        new AuthenticationHeaderValue("Bearer", result.AccessToken);
                }));

            await GetUserInfo();

            IsSignedIn = true;
        }
        else
        {
            IsSignedIn = false;
        }
    }
    catch(Exception ex)
    {
        Debug.WriteLine(
            $"Failed to initialized graph client. Accounts in the msal cache: {currentAccounts.Count()}. See exception message for details: {ex.Message}");
    }
}
```

<span data-ttu-id="7ca30-169">Ahora, actualice `SignIn` la función en **app.Xaml.CS** para llamar a esta función en `GetUserInfo`lugar de.</span><span class="sxs-lookup"><span data-stu-id="7ca30-169">Now update the `SignIn` function in **App.xaml.cs** to call this function instead of `GetUserInfo`.</span></span> <span data-ttu-id="7ca30-170">Quite lo siguiente de la `SignIn` función.</span><span class="sxs-lookup"><span data-stu-id="7ca30-170">Remove the following from the `SignIn` function.</span></span>

```cs
await GetUserInfo();

IsSignedIn = true;
```

<span data-ttu-id="7ca30-171">Agregue lo siguiente al final de la `SignIn` función.</span><span class="sxs-lookup"><span data-stu-id="7ca30-171">Add the following to the end of the `SignIn` function.</span></span>

```cs
await InitializeGraphClientAsync();
```

<span data-ttu-id="7ca30-172">Ahora, actualice `GetUserInfo` la función para obtener los detalles del usuario de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="7ca30-172">Now update the `GetUserInfo` function to get the user's details from the Microsoft Graph.</span></span> <span data-ttu-id="7ca30-173">Reemplace la función `GetUserInfo` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="7ca30-173">Replace the existing `GetUserInfo` function with the following.</span></span>

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

<span data-ttu-id="7ca30-174">Si guarda los cambios y ejecuta la aplicación ahora, después de iniciar sesión, la interfaz de usuario se actualizará con el nombre para mostrar y la dirección de correo electrónico del usuario.</span><span class="sxs-lookup"><span data-stu-id="7ca30-174">If you save your changes and run the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
