<!-- markdownlint-disable MD002 MD041 -->

1. Abra Visual Studio y seleccione **crear un nuevo proyecto**.

1. En el cuadro de diálogo **crear un nuevo proyecto** , elija **aplicación móvil (Xamarin. Forms)** y, a continuación, seleccione **siguiente**.

    ![Cuadro de diálogo Crear nuevo proyecto de Visual Studio 2019](images/new-project-dialog.png)

1. En el cuadro de diálogo **configurar un nuevo proyecto** `GraphTutorial` , escriba para el nombre del **proyecto** y el nombre de la **solución**y, después, seleccione **crear**.

    > [!IMPORTANT]
    > Asegúrese de que escribe exactamente el mismo nombre para el proyecto de Visual Studio que se especifica en estas instrucciones de la práctica. El nombre del proyecto de Visual Studio se convierte en parte del espacio de nombres en el código. El código incluido en estas instrucciones depende del espacio de nombres que coincida con el nombre de proyecto de Visual Studio especificado en estas instrucciones. Si usa un nombre de proyecto diferente, el código no se compilará a menos que ajuste todos los espacios de nombres para que se correspondan con el nombre del proyecto de Visual Studio que ha especificado al crear el proyecto.

    ![Cuadro de diálogo Configurar nuevo proyecto de Visual Studio 2019](images/configure-new-project-dialog.png)

1. En el cuadro de diálogo **nueva aplicación para varias plataformas** , seleccione la plantilla **en blanco** y, a continuación, seleccione las plataformas que desea crear en **plataformas**. Seleccione **Aceptar** para crear la solución.

    ![Visual Studio 2019 cuadro de diálogo Nueva aplicación para varias plataformas](images/new-cross-platform-app-dialog.png)

## <a name="install-packages"></a>Instalar paquetes

Antes de continuar, instale algunos paquetes NuGet adicionales que usará más adelante.

- [Microsoft. Identity. Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) para administrar la autenticación y la administración de tokens de Azure ad.
- [Microsoft. Graph](https://www.nuget.org/packages/Microsoft.Graph/) para realizar llamadas a Microsoft Graph.

1. Seleccione **herramientas > el administrador de paquetes de NuGet > consola del administrador de paquetes**.

1. En la consola del administrador de paquetes, escriba los siguientes comandos.

    ```Powershell
    Install-Package Microsoft.Identity.Client -Version 4.10.0 -Project GraphTutorial
    Install-Package Microsoft.Identity.Client -Version 4.10.0 -Project GraphTutorial.Android
    Install-Package Microsoft.Identity.Client -Version 4.10.0 -Project GraphTutorial.iOS
    Install-Package Microsoft.Graph -Version 3.0.1 -Project GraphTutorial
    ```

## <a name="design-the-app"></a>Diseñar la aplicación

Para empezar, actualice `App` la clase para agregar variables para realizar un seguimiento del estado de autenticación y del usuario que ha iniciado sesión.

1. En el **Explorador de soluciones**, expanda el proyecto **GraphTutorial** y, a continuación, expanda el archivo **app. Xaml** . Abra el archivo **app.Xaml.CS** y agregue las siguientes `using` instrucciones en la parte superior del archivo.

    ```csharp
    using System.ComponentModel;
    using System.IO;
    using System.Reflection;
    using System.Threading.Tasks;
    ```

1. Agregue la `INotifyPropertyChanged` interfaz a la declaración de clase.

    ```csharp
    public partial class App : Application, INotifyPropertyChanged
    ```

1. Agregue las siguientes propiedades a la `App` clase.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GlobalPropertiesSnippet":::

1. Agregue las siguientes funciones a la `App` clase. Las `SignIn`funciones `SignOut`, y `GetUserInfo` son solo marcadores de posición por ahora.

    ```csharp
    public async Task SignIn()
    {
        await GetUserInfo();

        IsSignedIn = true;
    }

    public async Task SignOut()
    {
        UserPhoto = null;
        UserName = string.Empty;
        UserEmail = string.Empty;
        IsSignedIn = false;
    }

    private async Task GetUserInfo()
    {
        UserPhoto = ImageSource.FromStream(() => GetUserPhoto());
        UserName = "Adele Vance";
        UserEmail = "adelev@contoso.com";
    }

    private Stream GetUserPhoto()
    {
        // Return the default photo
        return Assembly.GetExecutingAssembly().GetManifestResourceStream("GraphTutorial.no-profile-pic.png");
    }
    ```

1. La `GetUserPhoto` función devuelve una foto predeterminada por ahora. Puede proporcionar su propio archivo o puede descargar el que se usa en la muestra desde [GitHub](https://github.com/microsoftgraph/msgraph-training-xamarin/blob/master/tutorial/images/no-profile-pic.png). Si usa su propio archivo, cambie el nombre a **no-Profile-PIC. png**.

1. Copie el archivo en el directorio **./GraphTutorial/GraphTutorial** .

1. Haga clic con el botón secundario en el archivo en el **Explorador de soluciones** y seleccione **propiedades**. En la ventana **propiedades** , cambie el valor de la **acción de generación** a **recurso incrustado**.

    ![Captura de pantalla de la ventana Propiedades para el archivo PNG](./images/png-file-properties.png)

### <a name="app-navigation"></a>Navegación por la aplicación

En esta sección, cambiará la Página principal de la aplicación a una [Página principal-detalle](/xamarin/xamarin-forms/app-fundamentals/navigation/master-detail-page). Se proporcionará un menú de navegación para cambiar entre las vistas de la aplicación.

1. Abra el archivo **mainpage. Xaml** en el proyecto **GraphTutorial** y reemplace su contenido por lo siguiente.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/MainPage.xaml":::

#### <a name="implement-the-menu"></a>Implementar el menú

1. Haga clic con el botón derecho en el proyecto **GraphTutorial** y seleccione **Agregar**y, a continuación, **nueva carpeta**. Asigne un nombre `Models`a la carpeta.

1. Haga clic con el botón secundario en la carpeta **modelos** y seleccione **Agregar**y, a continuación, **clase...**. Asigne un nombre `NavMenuItem` a la clase y seleccione **Agregar**.

1. Abra el archivo **NavMenuItem.CS** y reemplace el contenido por lo siguiente.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/NavMenuItem.cs" id="NavMenuItemSnippet":::

1. Haga clic con el botón derecho en el proyecto **GraphTutorial** y seleccione **Agregar**y, a continuación, **nuevo elemento.**... Elija **Página de contenido** y asigne un `MenuPage`nombre a la página. Elija **Agregar**.

1. Abra el archivo **MenuPage. Xaml** y reemplace su contenido por lo siguiente.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/MenuPage.xaml":::

1. Expanda **MenuPage. Xaml** en el **Explorador de soluciones** y abra el archivo **MenuPage.Xaml.CS** . Reemplace su contenido por lo siguiente.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/MenuPage.xaml.cs" id="MenuPageSnippet":::

    > [!NOTE]
    > Visual Studio generará un informe de errores en **MenuPage.Xaml.CS**. Estos errores se resolverán en un paso posterior.

#### <a name="implement-the-welcome-page"></a>Implementar la página de bienvenida

1. Haga clic con el botón derecho en el proyecto **GraphTutorial** y seleccione **Agregar**y, a continuación, **nuevo elemento.**... Elija **Página de contenido** y asigne un `WelcomePage`nombre a la página. Elija **Agregar**. Abra el archivo **WelcomePage. Xaml** y reemplace su contenido por lo siguiente.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/WelcomePage.xaml":::

1. Expanda **WelcomePage. Xaml** en el **Explorador de soluciones** y abra el archivo **WelcomePage.Xaml.CS** . Agregue la siguiente función a la clase `WelcomePage`.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/WelcomePage.xaml.cs" id="OnSignInSnippet":::

#### <a name="add-calendar-page"></a>Agregar página de calendario

Ahora, agregue una página de calendario. Esto será simplemente un marcador de posición por ahora.

1. Haga clic con el botón derecho en el proyecto **GraphTutorial** y seleccione **Agregar**y, a continuación, **nuevo elemento.**... Elija **Página de contenido** y asigne un `CalendarPage`nombre a la página. Elija **Agregar**.

#### <a name="update-mainpage-code-behind"></a>Actualizar código subyacente de la MainPage

Ahora que todas las páginas están en su ubicación, actualiza el código subyacente para **mainpage. Xaml**.

1. Expanda **mainpage. Xaml** en el **Explorador de soluciones** y abra el archivo **mainpage.Xaml.CS** y reemplace todo el contenido por lo siguiente.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/MainPage.xaml.cs" id="MainPageSnippet":::

1. Guarde todos los cambios. Haz clic con el botón derecho en el proyecto que quieras ejecutar (Android, iOS o UWP) y selecciona **establecer como proyecto de inicio**. Presione **F5** o seleccione **depurar > iniciar depuración** en Visual Studio.

    ![Capturas de pantallas de las versiones de Android, iOS y UWP de la aplicación](./images/welcome-page.png)
