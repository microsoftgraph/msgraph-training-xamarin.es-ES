<!-- markdownlint-disable MD002 MD041 -->

1. Abra Visual Studio y seleccione **Crear un nuevo proyecto.**

1. En el cuadro de diálogo Crear un **nuevo proyecto,** elija **Aplicación móvil (Xamarin.Forms)** y, a continuación, **seleccione Siguiente**.

    ![Visual Studio 2019 crear nuevo cuadro de diálogo de proyecto](images/new-project-dialog.png)

1. En el **cuadro de diálogo** Configurar un nuevo proyecto, escriba el nombre del proyecto y el nombre de la solución y, a continuación, seleccione `GraphTutorial`  **Crear**. 

    > [!IMPORTANT]
    > Asegúrese de escribir exactamente el mismo nombre para el proyecto Visual Studio que se especifica en estas instrucciones de laboratorio. El Visual Studio nombre del proyecto pasa a formar parte del espacio de nombres en el código. El código dentro de estas instrucciones depende del espacio de nombres que coincida Visual Studio nombre del proyecto especificado en estas instrucciones. Si usa un nombre de proyecto diferente, el código no se compilará a menos que ajuste todos los espacios de nombres para que coincidan con el nombre de proyecto de Visual Studio que escriba al crear el proyecto.

    ![Visual Studio 2019 configurar el cuadro de diálogo de nuevo proyecto](images/configure-new-project-dialog.png)

1. En el **cuadro de diálogo Nueva** aplicación multiplataforma, seleccione **la** plantilla en blanco y seleccione las plataformas que desea crear en **Plataformas.** Seleccione **Aceptar** para crear la solución.

    ![Visual Studio diálogo de la nueva aplicación multiplataforma de 2019](images/new-cross-platform-app-dialog.png)

## <a name="install-packages"></a>Instalar paquetes

Antes de seguir adelante, instala algunos paquetes NuGet adicionales que usarás más adelante.

- [Microsoft.Identity.Client para controlar](https://www.nuget.org/packages/Microsoft.Identity.Client/) la autenticación de Azure AD y la administración de tokens.
- [Microsoft.Graph para](https://www.nuget.org/packages/Microsoft.Graph/) realizar llamadas a Microsoft Graph.
- [TimeZoneConverter para](https://www.nuget.org/packages/TimeZoneConverter/) controlar zonas horarias multiplataforma.

1. Seleccione **Herramientas > Consola de Administrador de paquetes > Administrador de paquetes NuGet**.

1. En la Administrador de paquetes, escriba los siguientes comandos.

    ```Powershell
    Install-Package Microsoft.Identity.Client -Version 4.24.0 -Project GraphTutorial
    Install-Package Microsoft.Identity.Client -Version 4.24.0 -Project GraphTutorial.Android
    Install-Package Microsoft.Identity.Client -Version 4.24.0 -Project GraphTutorial.iOS
    Install-Package Microsoft.Graph -Version 3.21.0 -Project GraphTutorial
    Install-Package TimeZoneConverter -Version 3.3.0 -Project GraphTutorial
    ```

## <a name="design-the-app"></a>Diseñar la aplicación

Empiece por actualizar la `App` clase para agregar variables para realizar un seguimiento del estado de autenticación y el usuario que ha iniciado sesión.

1. En **el Explorador de** soluciones, expanda el proyecto **GraphTutorial** y, a continuación, expanda **el archivo App.xaml.** Abra el **App.xaml.cs** archivo y agregue las siguientes instrucciones en `using` la parte superior del archivo.

    ```csharp
    using System.ComponentModel;
    using System.IO;
    using System.Reflection;
    using System.Threading.Tasks;
    ```

1. Agrega la `INotifyPropertyChanged` interfaz a la declaración de clase.

    ```csharp
    public partial class App : Application, INotifyPropertyChanged
    ```

1. Agregue las siguientes propiedades a la `App` clase.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GlobalPropertiesSnippet":::

1. Agregue las siguientes funciones a la `App` clase. Las `SignIn` funciones , y son solo `SignOut` `GetUserInfo` marcadores de posición por ahora.

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

1. Por `GetUserPhoto` ahora, la función devuelve una foto predeterminada. Puede proporcionar su propio archivo o puede descargar el que se usa en el ejemplo desde [GitHub.](https://github.com/microsoftgraph/msgraph-training-xamarin/blob/master/tutorial/images/no-profile-pic.png) Si usa su propio archivo, cambie el nombre a **no-profile-pic.png**.

1. Copie el archivo en el directorio **./GraphTutorial/GraphTutorial.**

1. Haga clic con el botón secundario en el archivo en **el Explorador de soluciones** y seleccione **Propiedades**. En la **ventana** Propiedades, cambie el valor de **Acción de compilación** a **recurso incrustado.**

    ![Captura de pantalla de la ventana Propiedades del archivo PNG](./images/png-file-properties.png)

### <a name="app-navigation"></a>Navegación de la aplicación

En esta sección, cambiará la página principal de la aplicación a una [página de maestro y detalles.](/xamarin/xamarin-forms/app-fundamentals/navigation/master-detail-page) Esto proporcionará un menú de navegación para cambiar entre la vista de la aplicación.

1. Abra el **archivo MainPage.xaml** en el **proyecto GraphTutorial** y reemplace su contenido por lo siguiente.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/MainPage.xaml" id="MainPageXamlSnippet":::

#### <a name="implement-the-menu"></a>Implementar el menú

1. Haga clic con el botón secundario **en el proyecto GraphTutorial** y seleccione **Agregar** y, a continuación, **Nueva carpeta.** Asigne un nombre a la carpeta `Models` .

1. Haga clic con el botón secundario en la carpeta **Modelos** y **seleccione Agregar** y, a continuación, **Clase...**. Asigne un nombre a `NavMenuItem` la clase y seleccione **Agregar**.

1. Abra el **NavMenuItem.cs** archivo y reemplace su contenido por lo siguiente.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/NavMenuItem.cs" id="NavMenuItemSnippet":::

1. Haga clic con el botón secundario **en el proyecto GraphTutorial** y seleccione **Agregar** y, a continuación, **Nuevo elemento...**. Elija **Página de contenido** y asigne un nombre a la `MenuPage` página. Seleccione **Agregar**.

1. Abre el **archivo MenuPage.xaml** y reemplaza su contenido por lo siguiente.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/MenuPage.xaml" id="MenuPageXamlSnippet":::

1. Expande **MenuPage.xaml** en **el Explorador de** soluciones y abre **MenuPage.xaml.cs** archivo. Reemplace su contenido por lo siguiente.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/MenuPage.xaml.cs" id="MenuPageSnippet":::

    > [!NOTE]
    > Visual Studio informe de errores en **MenuPage.xaml.cs**. Estos errores se resolverán en un paso posterior.

#### <a name="implement-the-welcome-page"></a>Implementar la página principal

1. Haga clic con el botón secundario **en el proyecto GraphTutorial** y seleccione **Agregar** y, a continuación, **Nuevo elemento...**. Elija **Página de contenido** y asigne un nombre a la `WelcomePage` página. Seleccione **Agregar**. Abre el **archivo WelcomePage.xaml** y reemplaza su contenido por lo siguiente.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/WelcomePage.xaml" id="WelcomePageXamlSnippet":::

1. Expande **WelcomePage.xaml** en **el Explorador de** soluciones y abre **WelcomePage.xaml.cs** archivo. Agregue la siguiente función a la clase `WelcomePage`.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/WelcomePage.xaml.cs" id="OnSignInSnippet":::

#### <a name="add-calendar-and-new-event-pages"></a>Agregar páginas de calendario y nuevos eventos

Ahora agregue una página de calendario y una nueva página de eventos. Por ahora, solo serán marcadores de posición.

1. Haga clic con el botón secundario **en el proyecto GraphTutorial** y seleccione **Agregar** y, a continuación, **Nuevo elemento...**. Elija **Página de contenido** y asigne un nombre a la `CalendarPage` página. Seleccione **Agregar**.

1. Haga clic con el botón secundario **en el proyecto GraphTutorial** y seleccione **Agregar** y, a continuación, **Nuevo elemento...**. Elija **Página de contenido** y asigne un nombre a la `NewEventPage` página. Seleccione **Agregar**.

#### <a name="update-mainpage-code-behind"></a>Actualizar código subyacente de MainPage

Ahora que todas las páginas están en su lugar, actualice el código subyacente para **MainPage.xaml**.

1. Expande **MainPage.xaml** en el **Explorador** de soluciones y abre **el archivo MainPage.xaml.cs** y reemplaza todo su contenido por lo siguiente.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/MainPage.xaml.cs" id="MainPageSnippet":::

1. Guarde todos los cambios. Haz clic con el botón derecho en el proyecto que quieras ejecutar (Android, iOS o UWP) y selecciona **Establecer como proyecto de inicio.** Presione **F5** o seleccione **Depurar > Iniciar depuración** en Visual Studio.

    ![Capturas de pantalla de las versiones de Android, iOS y UWP de la aplicación](./images/welcome-page.png)
