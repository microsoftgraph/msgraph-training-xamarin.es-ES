<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD. Esto es necesario para obtener el token de acceso de OAuth necesario para llamar a Microsoft Graph. En este paso, integrará la [biblioteca de autenticación de Microsoft para .net (MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) en la aplicación.

1. En el **Explorador de soluciones**, expanda el proyecto **GraphTutorial** y haga clic con el botón secundario en la carpeta **Models** . Seleccione **agregar > clase...**. Asigne un nombre `OAuthSettings` a la clase y seleccione **Agregar**.

1. Abra el archivo **OAuthSettings.CS** y reemplace el contenido por lo siguiente.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/OAuthSettings.cs.example":::

1. Reemplace `YOUR_APP_ID_HERE` por el identificador de la aplicación del registro de la aplicación.

    > [!IMPORTANT]
    > Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el `OAuthSettings.cs` archivo del control de código fuente para evitar la pérdida inadvertida del identificador de la aplicación.

## <a name="implement-sign-in"></a>Implementar el inicio de sesión

1. Abra el archivo **app.Xaml.CS** en el proyecto **GraphTutorial** y agregue las siguientes `using` instrucciones en la parte superior del archivo.

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;
    using System.Diagnostics;
    using System.Linq;
    using System.Net.Http.Headers;
    ```

1. Cambie la línea de declaración de clase de **aplicación** para resolver el conflicto de nombres de la **aplicación**.

    ```csharp
    public partial class App : Xamarin.Forms.Application, INotifyPropertyChanged
    ```

1. Agregue las siguientes propiedades a la `App` clase.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AuthPropertiesSnippet":::z

1. A continuación, cree un `PublicClientApplication` nuevo en el constructor de `App` la clase.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="AppConstructorSnippet" highlight="5-14":::

1. Actualice la `SignIn` función para que use `PublicClientApplication` el para obtener un token de acceso. Agregue el siguiente código encima de `await GetUserInfo();` la línea.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GetTokenSnippet":::

    Este código primero intenta obtener un token de acceso en modo silencioso. Si la información de un usuario ya se encuentra en la memoria caché de la aplicación (por ejemplo, si el usuario ha cerrado la aplicación anteriormente sin cerrar sesión), esto se realizará correctamente y no hay motivo para preguntar al usuario. Si no hay información de un usuario en la memoria caché, la `AcquireTokenSilent().ExecuteAsync()` función inicia una `MsalUiRequiredException`. En este caso, el código llama a la función interactiva para obtener un `AcquireTokenInteractive`token,.

1. Actualice la `SignOut` función para quitar la información del usuario de la memoria caché. Agregue el código siguiente al principio de la `SignOut` función.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="RemoveAccountSnippet":::

### <a name="update-android-project-to-enable-sign-in"></a>Actualizar el proyecto de Android para habilitar el inicio de sesión

Cuando se usa en un proyecto de Xamarin Android, la biblioteca de autenticación de Microsoft tiene algunos [requisitos específicos de Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics).

1. En el proyecto de **GraphTutorial. Android** , expanda la carpeta **propiedades** y Abra **AndroidManifest. XML**. Si está usando Visual Studio para Mac, control haga clic en **AndroidManifest. XML** y elija **abrir con**y, a continuación, **Editor de código fuente**. Reemplace todo el contenido por lo siguiente.

    :::code language="xml" source="../demo/GraphTutorial/GraphTutorial.Android/Properties/AndroidManifest.xml":::

1. Abra **MainActivity.CS** y agregue las siguientes `using` instrucciones en la parte superior del archivo.

    ```csharp
    using Android.Content;
    using Microsoft.Identity.Client;
    ```

1. Invalide la `OnActivityResult` función para pasar el control a la biblioteca de MSAL. Agregue lo siguiente a la `MainActivity` clase.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.Android/MainActivity.cs" id="OnActivityResultSnippet":::

1. En la `OnCreate` función, agregue la siguiente línea después de `LoadApplication(new App());` la línea.

    ```csharp
    App.AuthUIParent = this;
    ```

### <a name="update-ios-project-to-enable-sign-in"></a>Actualizar el proyecto de iOS para habilitar el inicio de sesión

> [!IMPORTANT]
> Como MSAL requiere el uso de un archivo de derechos. plist, debe configurar Visual Studio con su cuenta de desarrollador de Apple para habilitar el aprovisionamiento. Si está ejecutando este tutorial en el simulador de iPhone, debe agregar **derechos. plist** en el campo **derechos personalizados** en la configuración del proyecto de **GraphTutorial. iOS** , **compilación->firma del paquete de iOS**. Para obtener más información, consulte [aprovisionamiento de dispositivos para Xamarin. iOS](/xamarin/ios/get-started/installation/device-provisioning).

Cuando se usa en un proyecto de Xamarin iOS, la biblioteca de autenticación de Microsoft tiene algunos [requisitos específicos de iOS](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-iOS-specifics).

1. En el explorador de soluciones, expanda el proyecto **GraphTutorial. iOS** y, a continuación, abra el archivo **contitles. plist** .

1. Busque el derecho de la **cadena de claves** y seleccione **Habilitar cadena de claves**.

1. En **grupos de llaves**, agregue una entrada con el `com.companyname.GraphTutorial`formato.

    ![Captura de pantalla de la configuración de derechos de llaves](./images/enable-keychain-access.png)

1. Actualice el código en el proyecto **GraphTutorial. iOS** para controlar el redireccionamiento durante la autenticación. Abra el archivo **AppDelegate.CS** y agregue la siguiente `using` instrucción en la parte superior del archivo.

    ```csharp
    using Microsoft.Identity.Client;
    ```

1. Agregue la línea siguiente para `FinishedLaunching` que funcione justo antes `LoadApplication(new App());` de la línea.

    ```csharp
    // Specify the Keychain access group
    App.iOSKeychainSecurityGroup = NSBundle.MainBundle.BundleIdentifier;
    ```

1. Invalide la `OpenUrl` función para pasar la dirección URL a la biblioteca de MSAL. Agregue lo siguiente a la `AppDelegate` clase.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial.iOS/AppDelegate.cs" id="OpenUrlSnippet":::

## <a name="storing-the-tokens"></a>Almacenamiento de tokens

Cuando se usa la biblioteca de autenticación de Microsoft en un proyecto Xamarin, se aprovecha de forma predeterminada el almacenamiento seguro nativo para almacenar en caché los tokens. Para obtener más información, vea [serialización de la caché de tokens](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/token-cache-serialization) .

## <a name="test-sign-in"></a>Inicio de sesión de prueba

En este punto, si ejecuta la aplicación y pulsa el botón **iniciar sesión** , se le pedirá que inicie sesión. Si el inicio de sesión se realiza correctamente, debería ver el token de acceso impreso en el resultado del depurador.

![Captura de pantalla de la ventana de resultados en Visual Studio](./images/debugger-access-token.png)

## <a name="get-user-details"></a>Obtener detalles del usuario

1. Agregue una nueva función a la clase **App** para inicializar `GraphServiceClient`.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="InitializeGraphClientSnippet":::

1. Actualice la `SignIn` función en **app.Xaml.CS** para llamar a esta función en lugar `GetUserInfo`de. Quite lo siguiente de la `SignIn` función.

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
