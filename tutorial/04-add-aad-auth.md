<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="9bd68-101">En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="9bd68-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="9bd68-102">Esto es necesario para obtener el token de acceso de OAuth necesario para llamar a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="9bd68-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="9bd68-103">En este paso, integrará la [biblioteca de autenticación de Microsoft para .net (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="9bd68-103">In this step you will integrate the [Microsoft Authentication Library for .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) into the application.</span></span>

1. <span data-ttu-id="9bd68-104">En el **Explorador de soluciones**, expanda el proyecto **GraphTutorial** y haga clic con el botón secundario en la carpeta **Models** .</span><span class="sxs-lookup"><span data-stu-id="9bd68-104">In **Solution Explorer**, expand the **GraphTutorial** project and right-click the **Models** folder.</span></span> <span data-ttu-id="9bd68-105">Seleccione **agregar > clase...**. Asigne un nombre `OAuthSettings` a la clase y seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="9bd68-105">Select **Add > Class...**. Name the class `OAuthSettings` and select **Add**.</span></span>

1. <span data-ttu-id="9bd68-106">Abra el archivo **OAuthSettings.CS** y reemplace el contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="9bd68-106">Open the **OAuthSettings.cs** file and replace its contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/OAuthSettings.cs.example":::

1. <span data-ttu-id="9bd68-107">Reemplace `YOUR_APP_ID_HERE` por el identificador de la aplicación del registro de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="9bd68-107">Replace `YOUR_APP_ID_HERE` with the application ID from your app registration.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="9bd68-108">Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el `OAuthSettings.cs` archivo del control de código fuente para evitar la pérdida inadvertida del identificador de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="9bd68-108">If you're using source control such as git, now would be a good time to exclude the `OAuthSettings.cs` file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="9bd68-109">Implementar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="9bd68-109">Implement sign-in</span></span>

1. <span data-ttu-id="9bd68-110">Abra el archivo **app.Xaml.CS** en el proyecto **GraphTutorial** y agregue las siguientes `using` instrucciones en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="9bd68-110">Open the **App.xaml.cs** file in the **GraphTutorial** project, and add the following `using` statements to the top of the file.</span></span>

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;
    using System.Diagnostics;
    using System.Linq;
    using System.Net.Http.Headers;
    ```

1. <span data-ttu-id="9bd68-111">Cambie la línea de declaración de clase de **aplicación** para resolver el conflicto de nombres de la **aplicación**.</span><span class="sxs-lookup"><span data-stu-id="9bd68-111">Change the **App** class declaration line to resolve the name conflict for **Application**.</span></span>

    ```csharp
    public partial class App : Xamarin.Forms.Application, INotifyPropertyChanged
    ```

1. <span data-ttu-id="9bd68-112">Agregue las siguientes propiedades a la `App` clase.</span><span class="sxs-lookup"><span data-stu-id="9bd68-112">Add the following properties to the `App` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AuthPropertiesSnippet":::z

1. <span data-ttu-id="9bd68-113">A continuación, cree un `PublicClientApplication` nuevo en el constructor de `App` la clase.</span><span class="sxs-lookup"><span data-stu-id="9bd68-113">Next, create a new `PublicClientApplication` in the constructor of the `App` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AppConstructorSnippet" highlight="5-14":::

1. <span data-ttu-id="9bd68-114">Actualice la `SignIn` función para que use `PublicClientApplication` el para obtener un token de acceso.</span><span class="sxs-lookup"><span data-stu-id="9bd68-114">Update the `SignIn` function to use the `PublicClientApplication` to get an access token.</span></span> <span data-ttu-id="9bd68-115">Agregue el siguiente código encima de `await GetUserInfo();` la línea.</span><span class="sxs-lookup"><span data-stu-id="9bd68-115">Add the following code above the `await GetUserInfo();` line.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GetTokenSnippet":::

    <span data-ttu-id="9bd68-116">Este código primero intenta obtener un token de acceso en modo silencioso.</span><span class="sxs-lookup"><span data-stu-id="9bd68-116">This code first attempts to get an access token silently.</span></span> <span data-ttu-id="9bd68-117">Si la información de un usuario ya se encuentra en la memoria caché de la aplicación (por ejemplo, si el usuario ha cerrado la aplicación anteriormente sin cerrar sesión), esto se realizará correctamente y no hay motivo para preguntar al usuario.</span><span class="sxs-lookup"><span data-stu-id="9bd68-117">If a user's information is already in the app's cache (for example, if the user closed the app previously without signing out), this will succeed, and there's no reason to prompt the user.</span></span> <span data-ttu-id="9bd68-118">Si no hay información de un usuario en la memoria caché, la `AcquireTokenSilent().ExecuteAsync()` función inicia una `MsalUiRequiredException`.</span><span class="sxs-lookup"><span data-stu-id="9bd68-118">If there is not a user's information in the cache, the `AcquireTokenSilent().ExecuteAsync()` function throws an `MsalUiRequiredException`.</span></span> <span data-ttu-id="9bd68-119">En este caso, el código llama a la función interactiva para obtener un `AcquireTokenInteractive`token,.</span><span class="sxs-lookup"><span data-stu-id="9bd68-119">In this case, the code calls the interactive function to get a token, `AcquireTokenInteractive`.</span></span>

1. <span data-ttu-id="9bd68-120">Actualice la `SignOut` función para quitar la información del usuario de la memoria caché.</span><span class="sxs-lookup"><span data-stu-id="9bd68-120">Update the `SignOut` function to remove the user's information from the cache.</span></span> <span data-ttu-id="9bd68-121">Agregue el código siguiente al principio de la `SignOut` función.</span><span class="sxs-lookup"><span data-stu-id="9bd68-121">Add the following code to the beginning of the `SignOut` function.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="RemoveAccountSnippet":::

### <a name="update-android-project-to-enable-sign-in"></a><span data-ttu-id="9bd68-122">Actualizar el proyecto de Android para habilitar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="9bd68-122">Update Android project to enable sign-in</span></span>

<span data-ttu-id="9bd68-123">Cuando se usa en un proyecto de Xamarin Android, la biblioteca de autenticación de Microsoft tiene algunos [requisitos específicos de Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).</span><span class="sxs-lookup"><span data-stu-id="9bd68-123">When used in a Xamarin Android project, the Microsoft Authentication Library has a few [requirements specific to Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).</span></span>

1. <span data-ttu-id="9bd68-124">En el proyecto de **GraphTutorial. Android** , expanda la carpeta **propiedades** y Abra **AndroidManifest. XML**.</span><span class="sxs-lookup"><span data-stu-id="9bd68-124">In **GraphTutorial.Android** project, expand the **Properties** folder, then open **AndroidManifest.xml**.</span></span> <span data-ttu-id="9bd68-125">Si está usando Visual Studio para Mac, control haga clic en **AndroidManifest. XML** y elija **abrir con**y, a continuación, **Editor de código fuente**.</span><span class="sxs-lookup"><span data-stu-id="9bd68-125">If you're using Visual Studio for Mac, Control click **AndroidManifest.xml** and choose **Open With**, then **Source Code Editor**.</span></span> <span data-ttu-id="9bd68-126">Reemplace todo el contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="9bd68-126">Replace the entire contents with the following.</span></span>

    :::code language="xml" source="../demo/GraphTutorial/GraphTutorial.Android/Properties/AndroidManifest.xml":::

1. <span data-ttu-id="9bd68-127">Abra **MainActivity.CS** y agregue las siguientes `using` instrucciones en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="9bd68-127">Open **MainActivity.cs** and add the following `using` statements to the top of the file.</span></span>

    ```csharp
    using Android.Content;
    using Microsoft.Identity.Client;
    ```

1. <span data-ttu-id="9bd68-128">Invalide la `OnActivityResult` función para pasar el control a la biblioteca de MSAL.</span><span class="sxs-lookup"><span data-stu-id="9bd68-128">Override the `OnActivityResult` function to pass control to the MSAL library.</span></span> <span data-ttu-id="9bd68-129">Agregue lo siguiente a la `MainActivity` clase.</span><span class="sxs-lookup"><span data-stu-id="9bd68-129">Add the following to the `MainActivity` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.Android/MainActivity.cs" id="OnActivityResultSnippet":::

1. <span data-ttu-id="9bd68-130">En la `OnCreate` función, agregue la siguiente línea después de `LoadApplication(new App());` la línea.</span><span class="sxs-lookup"><span data-stu-id="9bd68-130">In the `OnCreate` function, add the following line after the `LoadApplication(new App());` line.</span></span>

    ```csharp
    App.AuthUIParent = this;
    ```

### <a name="update-ios-project-to-enable-sign-in"></a><span data-ttu-id="9bd68-131">Actualizar el proyecto de iOS para habilitar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="9bd68-131">Update iOS project to enable sign-in</span></span>

> [!IMPORTANT]
> <span data-ttu-id="9bd68-132">Como MSAL requiere el uso de un archivo de derechos. plist, debe configurar Visual Studio con su cuenta de desarrollador de Apple para habilitar el aprovisionamiento.</span><span class="sxs-lookup"><span data-stu-id="9bd68-132">Because MSAL requires use of an Entitlements.plist file, you must configure Visual Studio with your Apple developer account to enable provisioning.</span></span> <span data-ttu-id="9bd68-133">Si está ejecutando este tutorial en el simulador de iPhone, debe agregar **derechos. plist** en el campo **derechos personalizados** en la configuración del proyecto de **GraphTutorial. iOS** , **compilación->firma del paquete de iOS**.</span><span class="sxs-lookup"><span data-stu-id="9bd68-133">If you are running this tutorial in the iPhone simulator, you need to add **Entitlements.plist** in the **Custom Entitlements** field in **GraphTutorial.iOS** project's settings, **Build->iOS Bundle Signing**.</span></span> <span data-ttu-id="9bd68-134">Para obtener más información, consulte [aprovisionamiento de dispositivos para Xamarin. iOS](/xamarin/ios/get-started/installation/device-provisioning).</span><span class="sxs-lookup"><span data-stu-id="9bd68-134">For more information, see [Device provisioning for Xamarin.iOS](/xamarin/ios/get-started/installation/device-provisioning).</span></span>

<span data-ttu-id="9bd68-135">Cuando se usa en un proyecto de Xamarin iOS, la biblioteca de autenticación de Microsoft tiene algunos [requisitos específicos de iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).</span><span class="sxs-lookup"><span data-stu-id="9bd68-135">When used in a Xamarin iOS project, the Microsoft Authentication Library has a few [requirements specific to iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).</span></span>

1. <span data-ttu-id="9bd68-136">En el explorador de soluciones, expanda el proyecto **GraphTutorial. iOS** y, a continuación, abra el archivo **contitles. plist** .</span><span class="sxs-lookup"><span data-stu-id="9bd68-136">In Solution Explorer, expand the **GraphTutorial.iOS** project, then open the **Entitlements.plist** file.</span></span>

1. <span data-ttu-id="9bd68-137">Busque el derecho de la **cadena de claves** y seleccione **Habilitar cadena de claves**.</span><span class="sxs-lookup"><span data-stu-id="9bd68-137">Locate the **Keychain** entitlement, and select **Enable KeyChain**.</span></span>

1. <span data-ttu-id="9bd68-138">En **grupos de llaves**, agregue una entrada con el `com.companyname.GraphTutorial`formato.</span><span class="sxs-lookup"><span data-stu-id="9bd68-138">In **Keychain Groups**, add an entry with the format `com.companyname.GraphTutorial`.</span></span>

    ![Captura de pantalla de la configuración de derechos de llaves](./images/enable-keychain-access.png)

1. <span data-ttu-id="9bd68-140">Actualice el código en el proyecto **GraphTutorial. iOS** para controlar el redireccionamiento durante la autenticación.</span><span class="sxs-lookup"><span data-stu-id="9bd68-140">Update the code in the **GraphTutorial.iOS** project to handle the redirect during authentication.</span></span> <span data-ttu-id="9bd68-141">Abra el archivo **AppDelegate.CS** y agregue la siguiente `using` instrucción en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="9bd68-141">Open the **AppDelegate.cs** file and add the following `using` statement at the top of the file.</span></span>

    ```csharp
    using Microsoft.Identity.Client;
    ```

1. <span data-ttu-id="9bd68-142">Agregue la línea siguiente para `FinishedLaunching` que funcione justo antes `LoadApplication(new App());` de la línea.</span><span class="sxs-lookup"><span data-stu-id="9bd68-142">Add the following line to `FinishedLaunching` function just before the `LoadApplication(new App());` line.</span></span>

    ```csharp
    // Specify the Keychain access group
    App.iOSKeychainSecurityGroup = NSBundle.MainBundle.BundleIdentifier;
    ```

1. <span data-ttu-id="9bd68-143">Invalide la `OpenUrl` función para pasar la dirección URL a la biblioteca de MSAL.</span><span class="sxs-lookup"><span data-stu-id="9bd68-143">Override the `OpenUrl` function to pass the URL to the MSAL library.</span></span> <span data-ttu-id="9bd68-144">Agregue lo siguiente a la `AppDelegate` clase.</span><span class="sxs-lookup"><span data-stu-id="9bd68-144">Add the following to the `AppDelegate` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.iOS/AppDelegate.cs" id="OpenUrlSnippet":::

## <a name="storing-the-tokens"></a><span data-ttu-id="9bd68-145">Almacenamiento de tokens</span><span class="sxs-lookup"><span data-stu-id="9bd68-145">Storing the tokens</span></span>

<span data-ttu-id="9bd68-146">Cuando se usa la biblioteca de autenticación de Microsoft en un proyecto Xamarin, se aprovecha de forma predeterminada el almacenamiento seguro nativo para almacenar en caché los tokens.</span><span class="sxs-lookup"><span data-stu-id="9bd68-146">When the Microsoft Authentication Library is used in a Xamarin project, it takes advantage of the native secure storage to cache tokens by default.</span></span> <span data-ttu-id="9bd68-147">Para obtener más información, vea [serialización de la caché de tokens](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) .</span><span class="sxs-lookup"><span data-stu-id="9bd68-147">See [Token cache serialization](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) for more information.</span></span>

## <a name="test-sign-in"></a><span data-ttu-id="9bd68-148">Inicio de sesión de prueba</span><span class="sxs-lookup"><span data-stu-id="9bd68-148">Test sign-in</span></span>

<span data-ttu-id="9bd68-149">En este punto, si ejecuta la aplicación y pulsa el botón **iniciar sesión** , se le pedirá que inicie sesión.</span><span class="sxs-lookup"><span data-stu-id="9bd68-149">At this point if you run the application and tap the **Sign in** button, you are prompted to sign in.</span></span> <span data-ttu-id="9bd68-150">Si el inicio de sesión se realiza correctamente, debería ver el token de acceso impreso en el resultado del depurador.</span><span class="sxs-lookup"><span data-stu-id="9bd68-150">On successful sign in, you should see the access token printed into the debugger's output.</span></span>

![Captura de pantalla de la ventana de resultados en Visual Studio](./images/debugger-access-token.png)

## <a name="get-user-details"></a><span data-ttu-id="9bd68-152">Obtener detalles del usuario</span><span class="sxs-lookup"><span data-stu-id="9bd68-152">Get user details</span></span>

1. <span data-ttu-id="9bd68-153">Agregue una nueva función a la clase **App** para inicializar `GraphServiceClient`.</span><span class="sxs-lookup"><span data-stu-id="9bd68-153">Add a new function to the **App** class to initialize the `GraphServiceClient`.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="InitializeGraphClientSnippet":::

1. <span data-ttu-id="9bd68-154">Actualice la `SignIn` función en **app.Xaml.CS** para llamar a esta función en lugar `GetUserInfo`de.</span><span class="sxs-lookup"><span data-stu-id="9bd68-154">Update the `SignIn` function in **App.xaml.cs** to call this function instead of `GetUserInfo`.</span></span> <span data-ttu-id="9bd68-155">Quite lo siguiente de la `SignIn` función.</span><span class="sxs-lookup"><span data-stu-id="9bd68-155">Remove the following from the `SignIn` function.</span></span>

    ```csharp
    await GetUserInfo();

    IsSignedIn = true;
    ```

1. <span data-ttu-id="9bd68-156">Agregue lo siguiente al final de la `SignIn` función.</span><span class="sxs-lookup"><span data-stu-id="9bd68-156">Add the following to the end of the `SignIn` function.</span></span>

    ```csharp
    await InitializeGraphClientAsync();
    ```

1. <span data-ttu-id="9bd68-157">Actualice la `GetUserInfo` función para obtener los detalles del usuario de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="9bd68-157">Update the `GetUserInfo` function to get the user's details from the Microsoft Graph.</span></span> <span data-ttu-id="9bd68-158">Reemplace la función `GetUserInfo` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="9bd68-158">Replace the existing `GetUserInfo` function with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GetUserInfoSnippet":::

1. <span data-ttu-id="9bd68-159">Guarde los cambios y ejecute la aplicación.</span><span class="sxs-lookup"><span data-stu-id="9bd68-159">Save your changes and run the app.</span></span> <span data-ttu-id="9bd68-160">Después de iniciar sesión, la interfaz de usuario se actualiza con el nombre para mostrar y la dirección de correo electrónico del usuario.</span><span class="sxs-lookup"><span data-stu-id="9bd68-160">After sign-in the UI is updated with the user's display name and email address.</span></span>
