<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD. Esto es necesario para obtener el token de acceso OAuth necesario para llamar a Microsoft Graph. En este paso, integrará la Biblioteca de autenticación [de Microsoft para .NET (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) en la aplicación.

1. En **el Explorador de** soluciones, expanda el proyecto **GraphTutorial** y haga clic con el botón secundario en la **carpeta** Modelos. Seleccione **Agregar > clase...**. Asigne un nombre a `OAuthSettings` la clase y seleccione **Agregar**.

1. Abra el **OAuthSettings.cs** archivo y reemplace su contenido por lo siguiente.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/OAuthSettings.cs.example":::

1. Reemplace `YOUR_APP_ID_HERE` por el identificador de aplicación del registro de la aplicación.

    > [!IMPORTANT]
    > Si usas el control de código fuente como Git, ahora sería un buen momento para excluir el archivo del control de código fuente para evitar la pérdida involuntaria del identificador `OAuthSettings.cs` de la aplicación.

## <a name="implement-sign-in"></a>Implementar el inicio de sesión

1. Abra el **App.xaml.cs** en el proyecto **GraphTutorial** y agregue las siguientes instrucciones en `using` la parte superior del archivo.

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;
    using System.Diagnostics;
    using System.Linq;
    using System.Net.Http.Headers;
    using TimeZoneConverter;
    ```

1. Cambie la línea **de declaración** de clase de aplicación para resolver el conflicto de nombres de **Application**.

    ```csharp
    public partial class App : Xamarin.Forms.Application, INotifyPropertyChanged
    ```

1. Agregue las siguientes propiedades a la `App` clase.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AuthPropertiesSnippet":::z

1. A continuación, cree `PublicClientApplication` un nuevo en el constructor de la `App` clase.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AppConstructorSnippet" highlight="5-14":::

1. Actualice la `SignIn` función para usar para obtener un token de `PublicClientApplication` acceso. Agregue el siguiente código encima de la `await GetUserInfo();` línea.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GetTokenSnippet":::

    Este código primero intenta obtener un token de acceso de forma silenciosa. Si la información de un usuario ya está en la memoria caché de la aplicación (por ejemplo, si el usuario cerró la aplicación anteriormente sin cerrar sesión), se realizará correctamente y no hay ninguna razón para preguntar al usuario. Si no hay información de un usuario en la memoria caché, la `AcquireTokenSilent().ExecuteAsync()` función inicia un archivo `MsalUiRequiredException` . En este caso, el código llama a la función interactiva para obtener un `AcquireTokenInteractive` token.

1. Actualice la `SignOut` función para quitar la información del usuario de la memoria caché. Agregue el siguiente código al principio de la `SignOut` función.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="RemoveAccountSnippet":::

### <a name="update-android-project-to-enable-sign-in"></a>Actualizar el proyecto de Android para habilitar el inicio de sesión

Cuando se usa en un proyecto xamarin android, la biblioteca de autenticación de Microsoft tiene algunos [requisitos específicos para Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).

1. En **el proyecto GraphTutorial.Android,** expanda la carpeta **Propiedades** y, a continuación, **AndroidManifest.xml**. Si usas la versión Visual Studio para Mac, haz clic en **ControlAndroidManifest.xml** y elige Abrir **con** y, a continuación, editor de **código fuente.** Reemplace todo el contenido por lo siguiente.

    :::code language="xml" source="../demo/GraphTutorial/GraphTutorial.Android/Properties/AndroidManifest.xml":::

1. Abra **MainActivity.cs** y agregue las `using` siguientes instrucciones en la parte superior del archivo.

    ```csharp
    using Android.Content;
    using Microsoft.Identity.Client;
    ```

1. Invalide `OnActivityResult` la función para pasar el control a la biblioteca MSAL. Agregue lo siguiente a la `MainActivity` clase.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.Android/MainActivity.cs" id="OnActivityResultSnippet":::

1. En la `OnCreate` función, agregue la siguiente línea después de la `LoadApplication(new App());` línea.

    ```csharp
    App.AuthUIParent = this;
    ```

### <a name="update-ios-project-to-enable-sign-in"></a>Actualizar el proyecto de iOS para habilitar el inicio de sesión

> [!IMPORTANT]
> Dado que MSAL requiere el uso de un archivo Entitlements.plist, debes configurar Visual Studio con tu cuenta de desarrollador de Apple para habilitar el aprovisionamiento. Si está ejecutando este tutorial en el simulador de iPhone, debe agregar **Entitlements.plist** en el campo **Derechos personalizados** en la configuración del proyecto **GraphTutorial.iOS,** firma de lote **de iOS >** compilación. Para obtener más información, vea [El aprovisionamiento de dispositivos para Xamarin.iOS.](/xamarin/ios/get-started/installation/device-provisioning)

Cuando se usa en un proyecto de Xamarin para iOS, la biblioteca de autenticación de Microsoft tiene algunos [requisitos específicos de iOS.](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics)

1. En el Explorador de soluciones, expanda **el proyecto GraphTutorial.iOS** y, a continuación, abra el **archivo Entitlements.plist.**

1. Busque el **derecho de cadena de** claves y seleccione Habilitar **llave.**

1. En **Grupos de llaves,** agregue una entrada con el formato `com.companyname.GraphTutorial` .

    ![Captura de pantalla de la configuración de derechos de la cadena de claves](./images/enable-keychain-access.png)

1. Actualice el código del proyecto **GraphTutorial.iOS** para controlar el redireccionamiento durante la autenticación. Abra el **AppDelegate.cs** archivo y agregue la siguiente `using` instrucción en la parte superior del archivo.

    ```csharp
    using Microsoft.Identity.Client;
    ```

1. Agregue la siguiente línea para `FinishedLaunching` que funcione justo antes de la `LoadApplication(new App());` línea.

    ```csharp
    // Specify the Keychain access group
    App.iOSKeychainSecurityGroup = NSBundle.MainBundle.BundleIdentifier;
    ```

1. Invalide `OpenUrl` la función para pasar la dirección URL a la biblioteca MSAL. Agregue lo siguiente a la `AppDelegate` clase.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.iOS/AppDelegate.cs" id="OpenUrlSnippet":::

## <a name="storing-the-tokens"></a>Almacenar los tokens

Cuando se usa la biblioteca de autenticación de Microsoft en un proyecto xamarin, aprovecha el almacenamiento seguro nativo para almacenar tokens en caché de forma predeterminada. Vea [serialización de caché de tokens](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) para obtener más información.

## <a name="test-sign-in"></a>Probar el inicio de sesión

En este punto, si ejecuta la aplicación y pulsa el botón Iniciar **sesión,** se le pedirá que inicie sesión. Al iniciar sesión correctamente, debería ver el token de acceso impreso en la salida del depurador.

![Captura de pantalla de la ventana Salida en Visual Studio](./images/debugger-access-token.png)

## <a name="get-user-details"></a>Obtener detalles del usuario

1. Agregue una nueva función a la **clase App** para inicializar el `GraphServiceClient` archivo .

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="InitializeGraphClientSnippet":::

1. Actualice la `SignIn` función de **App.xaml.cs** para llamar a esta función en lugar de `GetUserInfo` . Quite lo siguiente de la `SignIn` función.

    ```csharp
    await GetUserInfo();

    IsSignedIn = true;
    ```

1. Agregue lo siguiente al final de la `SignIn` función.

    ```csharp
    await InitializeGraphClientAsync();
    ```

1. Actualice la `GetUserInfo` función para obtener los detalles del usuario de Microsoft Graph. Reemplace la función `GetUserInfo` existente por lo siguiente.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GetUserInfoSnippet":::

1. Guarde los cambios y ejecute la aplicación. Después de iniciar sesión, la interfaz de usuario se actualiza con el nombre para mostrar y la dirección de correo electrónico del usuario.
