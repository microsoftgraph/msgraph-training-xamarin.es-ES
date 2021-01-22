<!-- markdownlint-disable MD002 MD041 -->

1. <span data-ttu-id="37920-101">Abra Visual Studio y seleccione **Crear un nuevo proyecto.**</span><span class="sxs-lookup"><span data-stu-id="37920-101">Open Visual Studio, and select **Create a new project**.</span></span>

1. <span data-ttu-id="37920-102">En el cuadro de diálogo Crear un **nuevo proyecto,** elija **Aplicación móvil (Xamarin.Forms)** y, a continuación, **seleccione Siguiente**.</span><span class="sxs-lookup"><span data-stu-id="37920-102">In the **Create a new project** dialog, choose **Mobile App (Xamarin.Forms)**, then select **Next**.</span></span>

    ![Visual Studio 2019 crear nuevo cuadro de diálogo de proyecto](images/new-project-dialog.png)

1. <span data-ttu-id="37920-104">En el **cuadro de diálogo** Configurar un nuevo proyecto, escriba el nombre del proyecto y el nombre de la solución y, a continuación, seleccione `GraphTutorial`  **Crear**. </span><span class="sxs-lookup"><span data-stu-id="37920-104">In the **Configure a new project** dialog, enter `GraphTutorial` for the **Project name** and **Solution name**, then select **Create**.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="37920-105">Asegúrese de escribir exactamente el mismo nombre para el proyecto Visual Studio que se especifica en estas instrucciones de laboratorio.</span><span class="sxs-lookup"><span data-stu-id="37920-105">Ensure that you enter the exact same name for the Visual Studio Project that is specified in these lab instructions.</span></span> <span data-ttu-id="37920-106">El Visual Studio nombre del proyecto pasa a formar parte del espacio de nombres en el código.</span><span class="sxs-lookup"><span data-stu-id="37920-106">The Visual Studio Project name becomes part of the namespace in the code.</span></span> <span data-ttu-id="37920-107">El código dentro de estas instrucciones depende del espacio de nombres que coincida Visual Studio nombre del proyecto especificado en estas instrucciones.</span><span class="sxs-lookup"><span data-stu-id="37920-107">The code inside these instructions depends on the namespace matching the Visual Studio Project name specified in these instructions.</span></span> <span data-ttu-id="37920-108">Si usa un nombre de proyecto diferente, el código no se compilará a menos que ajuste todos los espacios de nombres para que coincidan con el nombre de proyecto de Visual Studio que escriba al crear el proyecto.</span><span class="sxs-lookup"><span data-stu-id="37920-108">If you use a different project name the code will not compile unless you adjust all the namespaces to match the Visual Studio Project name you enter when you create the project.</span></span>

    ![Visual Studio 2019 configurar el cuadro de diálogo de nuevo proyecto](images/configure-new-project-dialog.png)

1. <span data-ttu-id="37920-110">En el **cuadro de diálogo Nueva** aplicación multiplataforma, seleccione **la** plantilla en blanco y seleccione las plataformas que desea crear en **Plataformas.**</span><span class="sxs-lookup"><span data-stu-id="37920-110">In the **New Cross Platform App** dialog, select the **Blank** template, and select the platforms you want to build under **Platforms**.</span></span> <span data-ttu-id="37920-111">Seleccione **Aceptar** para crear la solución.</span><span class="sxs-lookup"><span data-stu-id="37920-111">Select **OK** to create the solution.</span></span>

    ![Visual Studio diálogo de la nueva aplicación multiplataforma de 2019](images/new-cross-platform-app-dialog.png)

## <a name="install-packages"></a><span data-ttu-id="37920-113">Instalar paquetes</span><span class="sxs-lookup"><span data-stu-id="37920-113">Install packages</span></span>

<span data-ttu-id="37920-114">Antes de seguir adelante, instala algunos paquetes NuGet adicionales que usarás más adelante.</span><span class="sxs-lookup"><span data-stu-id="37920-114">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="37920-115">[Microsoft.Identity.Client para controlar](https://www.nuget.org/packages/Microsoft.Identity.Client/) la autenticación de Azure AD y la administración de tokens.</span><span class="sxs-lookup"><span data-stu-id="37920-115">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) to handle Azure AD authentication and token management.</span></span>
- <span data-ttu-id="37920-116">[Microsoft.Graph para](https://www.nuget.org/packages/Microsoft.Graph/) realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="37920-116">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) for making calls to the Microsoft Graph.</span></span>
- <span data-ttu-id="37920-117">[TimeZoneConverter para](https://www.nuget.org/packages/TimeZoneConverter/) controlar zonas horarias multiplataforma.</span><span class="sxs-lookup"><span data-stu-id="37920-117">[TimeZoneConverter](https://www.nuget.org/packages/TimeZoneConverter/) for handling time zones cross-platform.</span></span>

1. <span data-ttu-id="37920-118">Seleccione **Herramientas > Consola de Administrador de paquetes > Administrador de paquetes NuGet**.</span><span class="sxs-lookup"><span data-stu-id="37920-118">Select **Tools > NuGet Package Manager > Package Manager Console**.</span></span>

1. <span data-ttu-id="37920-119">En la Administrador de paquetes, escriba los siguientes comandos.</span><span class="sxs-lookup"><span data-stu-id="37920-119">In the Package Manager Console, enter the following commands.</span></span>

    ```Powershell
    Install-Package Microsoft.Identity.Client -Version 4.24.0 -Project GraphTutorial
    Install-Package Microsoft.Identity.Client -Version 4.24.0 -Project GraphTutorial.Android
    Install-Package Microsoft.Identity.Client -Version 4.24.0 -Project GraphTutorial.iOS
    Install-Package Microsoft.Graph -Version 3.21.0 -Project GraphTutorial
    Install-Package TimeZoneConverter -Version 3.3.0 -Project GraphTutorial
    ```

## <a name="design-the-app"></a><span data-ttu-id="37920-120">Diseñar la aplicación</span><span class="sxs-lookup"><span data-stu-id="37920-120">Design the app</span></span>

<span data-ttu-id="37920-121">Empiece por actualizar la `App` clase para agregar variables para realizar un seguimiento del estado de autenticación y el usuario que ha iniciado sesión.</span><span class="sxs-lookup"><span data-stu-id="37920-121">Start by updating the `App` class to add variables to track the authentication state and the signed-in user.</span></span>

1. <span data-ttu-id="37920-122">En **el Explorador de** soluciones, expanda el proyecto **GraphTutorial** y, a continuación, expanda **el archivo App.xaml.**</span><span class="sxs-lookup"><span data-stu-id="37920-122">In **Solution Explorer**, expand the **GraphTutorial** project, then expand the **App.xaml** file.</span></span> <span data-ttu-id="37920-123">Abra el **App.xaml.cs** archivo y agregue las siguientes instrucciones en `using` la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="37920-123">Open the **App.xaml.cs** file and add the following `using` statements to the top of the file.</span></span>

    ```csharp
    using System.ComponentModel;
    using System.IO;
    using System.Reflection;
    using System.Threading.Tasks;
    ```

1. <span data-ttu-id="37920-124">Agrega la `INotifyPropertyChanged` interfaz a la declaración de clase.</span><span class="sxs-lookup"><span data-stu-id="37920-124">Add the `INotifyPropertyChanged` interface to the class declaration.</span></span>

    ```csharp
    public partial class App : Application, INotifyPropertyChanged
    ```

1. <span data-ttu-id="37920-125">Agregue las siguientes propiedades a la `App` clase.</span><span class="sxs-lookup"><span data-stu-id="37920-125">Add the following properties to the `App` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GlobalPropertiesSnippet":::

1. <span data-ttu-id="37920-126">Agregue las siguientes funciones a la `App` clase.</span><span class="sxs-lookup"><span data-stu-id="37920-126">Add the following functions to the `App` class.</span></span> <span data-ttu-id="37920-127">Las `SignIn` funciones , y son solo `SignOut` `GetUserInfo` marcadores de posición por ahora.</span><span class="sxs-lookup"><span data-stu-id="37920-127">The `SignIn`, `SignOut`, and `GetUserInfo` functions are just placeholders for now.</span></span>

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

1. <span data-ttu-id="37920-128">Por `GetUserPhoto` ahora, la función devuelve una foto predeterminada.</span><span class="sxs-lookup"><span data-stu-id="37920-128">The `GetUserPhoto` function returns a default photo for now.</span></span> <span data-ttu-id="37920-129">Puede proporcionar su propio archivo o puede descargar el que se usa en el ejemplo desde [GitHub.](https://github.com/microsoftgraph/msgraph-training-xamarin/blob/master/tutorial/images/no-profile-pic.png)</span><span class="sxs-lookup"><span data-stu-id="37920-129">You can either supply your own file, or you can download the one used in the sample from [GitHub](https://github.com/microsoftgraph/msgraph-training-xamarin/blob/master/tutorial/images/no-profile-pic.png).</span></span> <span data-ttu-id="37920-130">Si usa su propio archivo, cambie el nombre a **no-profile-pic.png**.</span><span class="sxs-lookup"><span data-stu-id="37920-130">If you use your own file, rename it to **no-profile-pic.png**.</span></span>

1. <span data-ttu-id="37920-131">Copie el archivo en el directorio **./GraphTutorial/GraphTutorial.**</span><span class="sxs-lookup"><span data-stu-id="37920-131">Copy the file to the **./GraphTutorial/GraphTutorial** directory.</span></span>

1. <span data-ttu-id="37920-132">Haga clic con el botón secundario en el archivo en **el Explorador de soluciones** y seleccione **Propiedades**.</span><span class="sxs-lookup"><span data-stu-id="37920-132">Right-click the file in **Solution Explorer** and select **Properties**.</span></span> <span data-ttu-id="37920-133">En la **ventana** Propiedades, cambie el valor de **Acción de compilación** a **recurso incrustado.**</span><span class="sxs-lookup"><span data-stu-id="37920-133">In the **Properties** window, change the value of **Build Action** to **Embedded resource**.</span></span>

    ![Captura de pantalla de la ventana Propiedades del archivo PNG](./images/png-file-properties.png)

### <a name="app-navigation"></a><span data-ttu-id="37920-135">Navegación de la aplicación</span><span class="sxs-lookup"><span data-stu-id="37920-135">App navigation</span></span>

<span data-ttu-id="37920-136">En esta sección, cambiará la página principal de la aplicación a una [página de maestro y detalles.](/xamarin/xamarin-forms/app-fundamentals/navigation/master-detail-page)</span><span class="sxs-lookup"><span data-stu-id="37920-136">In this section, you'll change the application's main page to a [Master-Detail page](/xamarin/xamarin-forms/app-fundamentals/navigation/master-detail-page).</span></span> <span data-ttu-id="37920-137">Esto proporcionará un menú de navegación para cambiar entre la vista de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="37920-137">This will provide a navigation menu to switch between view in the app.</span></span>

1. <span data-ttu-id="37920-138">Abra el **archivo MainPage.xaml** en el **proyecto GraphTutorial** y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="37920-138">Open the **MainPage.xaml** file in the **GraphTutorial** project and replace its contents with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/MainPage.xaml" id="MainPageXamlSnippet":::

#### <a name="implement-the-menu"></a><span data-ttu-id="37920-139">Implementar el menú</span><span class="sxs-lookup"><span data-stu-id="37920-139">Implement the menu</span></span>

1. <span data-ttu-id="37920-140">Haga clic con el botón secundario **en el proyecto GraphTutorial** y seleccione **Agregar** y, a continuación, **Nueva carpeta.**</span><span class="sxs-lookup"><span data-stu-id="37920-140">Right-click the **GraphTutorial** project and select **Add**, then **New Folder**.</span></span> <span data-ttu-id="37920-141">Asigne un nombre a la carpeta `Models` .</span><span class="sxs-lookup"><span data-stu-id="37920-141">Name the folder `Models`.</span></span>

1. <span data-ttu-id="37920-142">Haga clic con el botón secundario en la carpeta **Modelos** y **seleccione Agregar** y, a continuación, **Clase...**. Asigne un nombre a `NavMenuItem` la clase y seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="37920-142">Right-click the **Models** folder and select **Add**, then **Class...**. Name the class `NavMenuItem` and select **Add**.</span></span>

1. <span data-ttu-id="37920-143">Abra el **NavMenuItem.cs** archivo y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="37920-143">Open the **NavMenuItem.cs** file and replace its contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/NavMenuItem.cs" id="NavMenuItemSnippet":::

1. <span data-ttu-id="37920-144">Haga clic con el botón secundario **en el proyecto GraphTutorial** y seleccione **Agregar** y, a continuación, **Nuevo elemento...**. Elija **Página de contenido** y asigne un nombre a la `MenuPage` página.</span><span class="sxs-lookup"><span data-stu-id="37920-144">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `MenuPage`.</span></span> <span data-ttu-id="37920-145">Seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="37920-145">Select **Add**.</span></span>

1. <span data-ttu-id="37920-146">Abre el **archivo MenuPage.xaml** y reemplaza su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="37920-146">Open the **MenuPage.xaml** file and replace its contents with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/MenuPage.xaml" id="MenuPageXamlSnippet":::

1. <span data-ttu-id="37920-147">Expande **MenuPage.xaml** en **el Explorador de** soluciones y abre **MenuPage.xaml.cs** archivo.</span><span class="sxs-lookup"><span data-stu-id="37920-147">Expand **MenuPage.xaml** in **Solution Explorer** and open the **MenuPage.xaml.cs** file.</span></span> <span data-ttu-id="37920-148">Reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="37920-148">Replace its contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/MenuPage.xaml.cs" id="MenuPageSnippet":::

    > [!NOTE]
    > <span data-ttu-id="37920-149">Visual Studio informe de errores en **MenuPage.xaml.cs**.</span><span class="sxs-lookup"><span data-stu-id="37920-149">Visual Studio will report errors in **MenuPage.xaml.cs**.</span></span> <span data-ttu-id="37920-150">Estos errores se resolverán en un paso posterior.</span><span class="sxs-lookup"><span data-stu-id="37920-150">These errors will be resolved in a later step.</span></span>

#### <a name="implement-the-welcome-page"></a><span data-ttu-id="37920-151">Implementar la página principal</span><span class="sxs-lookup"><span data-stu-id="37920-151">Implement the welcome page</span></span>

1. <span data-ttu-id="37920-152">Haga clic con el botón secundario **en el proyecto GraphTutorial** y seleccione **Agregar** y, a continuación, **Nuevo elemento...**. Elija **Página de contenido** y asigne un nombre a la `WelcomePage` página.</span><span class="sxs-lookup"><span data-stu-id="37920-152">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `WelcomePage`.</span></span> <span data-ttu-id="37920-153">Seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="37920-153">Select **Add**.</span></span> <span data-ttu-id="37920-154">Abre el **archivo WelcomePage.xaml** y reemplaza su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="37920-154">Open the **WelcomePage.xaml** file and replace its contents with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/WelcomePage.xaml" id="WelcomePageXamlSnippet":::

1. <span data-ttu-id="37920-155">Expande **WelcomePage.xaml** en **el Explorador de** soluciones y abre **WelcomePage.xaml.cs** archivo.</span><span class="sxs-lookup"><span data-stu-id="37920-155">Expand **WelcomePage.xaml** in **Solution Explorer** and open the **WelcomePage.xaml.cs** file.</span></span> <span data-ttu-id="37920-156">Agregue la siguiente función a la clase `WelcomePage`.</span><span class="sxs-lookup"><span data-stu-id="37920-156">Add the following function to the `WelcomePage` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/WelcomePage.xaml.cs" id="OnSignInSnippet":::

#### <a name="add-calendar-and-new-event-pages"></a><span data-ttu-id="37920-157">Agregar páginas de calendario y nuevos eventos</span><span class="sxs-lookup"><span data-stu-id="37920-157">Add calendar and new event pages</span></span>

<span data-ttu-id="37920-158">Ahora agregue una página de calendario y una nueva página de eventos.</span><span class="sxs-lookup"><span data-stu-id="37920-158">Now add a calendar page and a new event page.</span></span> <span data-ttu-id="37920-159">Por ahora, solo serán marcadores de posición.</span><span class="sxs-lookup"><span data-stu-id="37920-159">These will just be placeholders for now.</span></span>

1. <span data-ttu-id="37920-160">Haga clic con el botón secundario **en el proyecto GraphTutorial** y seleccione **Agregar** y, a continuación, **Nuevo elemento...**. Elija **Página de contenido** y asigne un nombre a la `CalendarPage` página.</span><span class="sxs-lookup"><span data-stu-id="37920-160">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `CalendarPage`.</span></span> <span data-ttu-id="37920-161">Seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="37920-161">Select **Add**.</span></span>

1. <span data-ttu-id="37920-162">Haga clic con el botón secundario **en el proyecto GraphTutorial** y seleccione **Agregar** y, a continuación, **Nuevo elemento...**. Elija **Página de contenido** y asigne un nombre a la `NewEventPage` página.</span><span class="sxs-lookup"><span data-stu-id="37920-162">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `NewEventPage`.</span></span> <span data-ttu-id="37920-163">Seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="37920-163">Select **Add**.</span></span>

#### <a name="update-mainpage-code-behind"></a><span data-ttu-id="37920-164">Actualizar código subyacente de MainPage</span><span class="sxs-lookup"><span data-stu-id="37920-164">Update MainPage code-behind</span></span>

<span data-ttu-id="37920-165">Ahora que todas las páginas están en su lugar, actualice el código subyacente para **MainPage.xaml**.</span><span class="sxs-lookup"><span data-stu-id="37920-165">Now that all of the pages are in place, update the code-behind for **MainPage.xaml**.</span></span>

1. <span data-ttu-id="37920-166">Expande **MainPage.xaml** en el **Explorador** de soluciones y abre **el archivo MainPage.xaml.cs** y reemplaza todo su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="37920-166">Expand **MainPage.xaml** in **Solution Explorer** and open the **MainPage.xaml.cs** file and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/MainPage.xaml.cs" id="MainPageSnippet":::

1. <span data-ttu-id="37920-167">Guarde todos los cambios.</span><span class="sxs-lookup"><span data-stu-id="37920-167">Save all of your changes.</span></span> <span data-ttu-id="37920-168">Haz clic con el botón derecho en el proyecto que quieras ejecutar (Android, iOS o UWP) y selecciona **Establecer como proyecto de inicio.**</span><span class="sxs-lookup"><span data-stu-id="37920-168">Right-click the project that you want to run (Android, iOS, or UWP) and select **Set as StartUp Project**.</span></span> <span data-ttu-id="37920-169">Presione **F5** o seleccione **Depurar > Iniciar depuración** en Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="37920-169">Press **F5** or select **Debug > Start Debugging** in Visual Studio.</span></span>

    ![Capturas de pantalla de las versiones de Android, iOS y UWP de la aplicación](./images/welcome-page.png)
