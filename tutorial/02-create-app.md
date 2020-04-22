<!-- markdownlint-disable MD002 MD041 -->

1. <span data-ttu-id="98bcd-101">Abra Visual Studio y seleccione **crear un nuevo proyecto**.</span><span class="sxs-lookup"><span data-stu-id="98bcd-101">Open Visual Studio, and select **Create a new project**.</span></span>

1. <span data-ttu-id="98bcd-102">En el cuadro de diálogo **crear un nuevo proyecto** , elija **aplicación móvil (Xamarin. Forms)** y, a continuación, seleccione **siguiente**.</span><span class="sxs-lookup"><span data-stu-id="98bcd-102">In the **Create a new project** dialog, choose **Mobile App (Xamarin.Forms)**, then select **Next**.</span></span>

    ![Cuadro de diálogo Crear nuevo proyecto de Visual Studio 2019](images/new-project-dialog.png)

1. <span data-ttu-id="98bcd-104">En el cuadro de diálogo **configurar un nuevo proyecto** `GraphTutorial` , escriba para el nombre del **proyecto** y el nombre de la **solución**y, después, seleccione **crear**.</span><span class="sxs-lookup"><span data-stu-id="98bcd-104">In the **Configure a new project** dialog, enter `GraphTutorial` for the **Project name** and **Solution name**, then select **Create**.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="98bcd-105">Asegúrese de que escribe exactamente el mismo nombre para el proyecto de Visual Studio que se especifica en estas instrucciones de la práctica.</span><span class="sxs-lookup"><span data-stu-id="98bcd-105">Ensure that you enter the exact same name for the Visual Studio Project that is specified in these lab instructions.</span></span> <span data-ttu-id="98bcd-106">El nombre del proyecto de Visual Studio se convierte en parte del espacio de nombres en el código.</span><span class="sxs-lookup"><span data-stu-id="98bcd-106">The Visual Studio Project name becomes part of the namespace in the code.</span></span> <span data-ttu-id="98bcd-107">El código incluido en estas instrucciones depende del espacio de nombres que coincida con el nombre de proyecto de Visual Studio especificado en estas instrucciones.</span><span class="sxs-lookup"><span data-stu-id="98bcd-107">The code inside these instructions depends on the namespace matching the Visual Studio Project name specified in these instructions.</span></span> <span data-ttu-id="98bcd-108">Si usa un nombre de proyecto diferente, el código no se compilará a menos que ajuste todos los espacios de nombres para que se correspondan con el nombre del proyecto de Visual Studio que ha especificado al crear el proyecto.</span><span class="sxs-lookup"><span data-stu-id="98bcd-108">If you use a different project name the code will not compile unless you adjust all the namespaces to match the Visual Studio Project name you enter when you create the project.</span></span>

    ![Cuadro de diálogo Configurar nuevo proyecto de Visual Studio 2019](images/configure-new-project-dialog.png)

1. <span data-ttu-id="98bcd-110">En el cuadro de diálogo **nueva aplicación para varias plataformas** , seleccione la plantilla **en blanco** y, a continuación, seleccione las plataformas que desea crear en **plataformas**.</span><span class="sxs-lookup"><span data-stu-id="98bcd-110">In the **New Cross Platform App** dialog, select the **Blank** template, and select the platforms you want to build under **Platforms**.</span></span> <span data-ttu-id="98bcd-111">Seleccione **Aceptar** para crear la solución.</span><span class="sxs-lookup"><span data-stu-id="98bcd-111">Select **OK** to create the solution.</span></span>

    ![Visual Studio 2019 cuadro de diálogo Nueva aplicación para varias plataformas](images/new-cross-platform-app-dialog.png)

## <a name="install-packages"></a><span data-ttu-id="98bcd-113">Instalar paquetes</span><span class="sxs-lookup"><span data-stu-id="98bcd-113">Install packages</span></span>

<span data-ttu-id="98bcd-114">Antes de continuar, instale algunos paquetes NuGet adicionales que usará más adelante.</span><span class="sxs-lookup"><span data-stu-id="98bcd-114">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="98bcd-115">[Microsoft. Identity. Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) para administrar la autenticación y la administración de tokens de Azure ad.</span><span class="sxs-lookup"><span data-stu-id="98bcd-115">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) to handle Azure AD authentication and token management.</span></span>
- <span data-ttu-id="98bcd-116">[Microsoft. Graph](https://www.nuget.org/packages/Microsoft.Graph/) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="98bcd-116">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) for making calls to the Microsoft Graph.</span></span>

1. <span data-ttu-id="98bcd-117">Seleccione **herramientas > el administrador de paquetes de NuGet > consola del administrador de paquetes**.</span><span class="sxs-lookup"><span data-stu-id="98bcd-117">Select **Tools > NuGet Package Manager > Package Manager Console**.</span></span>

1. <span data-ttu-id="98bcd-118">En la consola del administrador de paquetes, escriba los siguientes comandos.</span><span class="sxs-lookup"><span data-stu-id="98bcd-118">In the Package Manager Console, enter the following commands.</span></span>

    ```Powershell
    Install-Package Microsoft.Identity.Client -Version 4.10.0 -Project GraphTutorial
    Install-Package Microsoft.Identity.Client -Version 4.10.0 -Project GraphTutorial.Android
    Install-Package Microsoft.Identity.Client -Version 4.10.0 -Project GraphTutorial.iOS
    Install-Package Microsoft.Graph -Version 3.0.1 -Project GraphTutorial
    ```

## <a name="design-the-app"></a><span data-ttu-id="98bcd-119">Diseñar la aplicación</span><span class="sxs-lookup"><span data-stu-id="98bcd-119">Design the app</span></span>

<span data-ttu-id="98bcd-120">Para empezar, actualice `App` la clase para agregar variables para realizar un seguimiento del estado de autenticación y del usuario que ha iniciado sesión.</span><span class="sxs-lookup"><span data-stu-id="98bcd-120">Start by updating the `App` class to add variables to track the authentication state and the signed-in user.</span></span>

1. <span data-ttu-id="98bcd-121">En el **Explorador de soluciones**, expanda el proyecto **GraphTutorial** y, a continuación, expanda el archivo **app. Xaml** .</span><span class="sxs-lookup"><span data-stu-id="98bcd-121">In **Solution Explorer**, expand the **GraphTutorial** project, then expand the **App.xaml** file.</span></span> <span data-ttu-id="98bcd-122">Abra el archivo **app.Xaml.CS** y agregue las siguientes `using` instrucciones en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="98bcd-122">Open the **App.xaml.cs** file and add the following `using` statements to the top of the file.</span></span>

    ```csharp
    using System.ComponentModel;
    using System.IO;
    using System.Reflection;
    using System.Threading.Tasks;
    ```

1. <span data-ttu-id="98bcd-123">Agregue la `INotifyPropertyChanged` interfaz a la declaración de clase.</span><span class="sxs-lookup"><span data-stu-id="98bcd-123">Add the `INotifyPropertyChanged` interface to the class declaration.</span></span>

    ```csharp
    public partial class App : Application, INotifyPropertyChanged
    ```

1. <span data-ttu-id="98bcd-124">Agregue las siguientes propiedades a la `App` clase.</span><span class="sxs-lookup"><span data-stu-id="98bcd-124">Add the following properties to the `App` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/App.xaml.cs" id="GlobalPropertiesSnippet":::

1. <span data-ttu-id="98bcd-125">Agregue las siguientes funciones a la `App` clase.</span><span class="sxs-lookup"><span data-stu-id="98bcd-125">Add the following functions to the `App` class.</span></span> <span data-ttu-id="98bcd-126">Las `SignIn`funciones `SignOut`, y `GetUserInfo` son solo marcadores de posición por ahora.</span><span class="sxs-lookup"><span data-stu-id="98bcd-126">The `SignIn`, `SignOut`, and `GetUserInfo` functions are just placeholders for now.</span></span>

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

1. <span data-ttu-id="98bcd-127">La `GetUserPhoto` función devuelve una foto predeterminada por ahora.</span><span class="sxs-lookup"><span data-stu-id="98bcd-127">The `GetUserPhoto` function returns a default photo for now.</span></span> <span data-ttu-id="98bcd-128">Puede proporcionar su propio archivo o puede descargar el que se usa en la muestra desde [GitHub](https://github.com/microsoftgraph/msgraph-training-xamarin/blob/master/tutorial/images/no-profile-pic.png).</span><span class="sxs-lookup"><span data-stu-id="98bcd-128">You can either supply your own file, or you can download the one used in the sample from [GitHub](https://github.com/microsoftgraph/msgraph-training-xamarin/blob/master/tutorial/images/no-profile-pic.png).</span></span> <span data-ttu-id="98bcd-129">Si usa su propio archivo, cambie el nombre a **no-Profile-PIC. png**.</span><span class="sxs-lookup"><span data-stu-id="98bcd-129">If you use your own file, rename it to **no-profile-pic.png**.</span></span>

1. <span data-ttu-id="98bcd-130">Copie el archivo en el directorio **./GraphTutorial/GraphTutorial** .</span><span class="sxs-lookup"><span data-stu-id="98bcd-130">Copy the file to the **./GraphTutorial/GraphTutorial** directory.</span></span>

1. <span data-ttu-id="98bcd-131">Haga clic con el botón secundario en el archivo en el **Explorador de soluciones** y seleccione **propiedades**.</span><span class="sxs-lookup"><span data-stu-id="98bcd-131">Right-click the file in **Solution Explorer** and select **Properties**.</span></span> <span data-ttu-id="98bcd-132">En la ventana **propiedades** , cambie el valor de la **acción de generación** a **recurso incrustado**.</span><span class="sxs-lookup"><span data-stu-id="98bcd-132">In the **Properties** window, change the value of **Build Action** to **Embedded resource**.</span></span>

    ![Captura de pantalla de la ventana Propiedades para el archivo PNG](./images/png-file-properties.png)

### <a name="app-navigation"></a><span data-ttu-id="98bcd-134">Navegación por la aplicación</span><span class="sxs-lookup"><span data-stu-id="98bcd-134">App navigation</span></span>

<span data-ttu-id="98bcd-135">En esta sección, cambiará la Página principal de la aplicación a una [Página principal-detalle](/xamarin/xamarin-forms/app-fundamentals/navigation/master-detail-page).</span><span class="sxs-lookup"><span data-stu-id="98bcd-135">In this section, you'll change the application's main page to a [Master-Detail page](/xamarin/xamarin-forms/app-fundamentals/navigation/master-detail-page).</span></span> <span data-ttu-id="98bcd-136">Se proporcionará un menú de navegación para cambiar entre las vistas de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="98bcd-136">This will provide a navigation menu to switch between view in the app.</span></span>

1. <span data-ttu-id="98bcd-137">Abra el archivo **mainpage. Xaml** en el proyecto **GraphTutorial** y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="98bcd-137">Open the **MainPage.xaml** file in the **GraphTutorial** project and replace its contents with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/MainPage.xaml":::

#### <a name="implement-the-menu"></a><span data-ttu-id="98bcd-138">Implementar el menú</span><span class="sxs-lookup"><span data-stu-id="98bcd-138">Implement the menu</span></span>

1. <span data-ttu-id="98bcd-139">Haga clic con el botón derecho en el proyecto **GraphTutorial** y seleccione **Agregar**y, a continuación, **nueva carpeta**.</span><span class="sxs-lookup"><span data-stu-id="98bcd-139">Right-click the **GraphTutorial** project and select **Add**, then **New Folder**.</span></span> <span data-ttu-id="98bcd-140">Asigne un nombre `Models`a la carpeta.</span><span class="sxs-lookup"><span data-stu-id="98bcd-140">Name the folder `Models`.</span></span>

1. <span data-ttu-id="98bcd-141">Haga clic con el botón secundario en la carpeta **modelos** y seleccione **Agregar**y, a continuación, **clase...**. Asigne un nombre `NavMenuItem` a la clase y seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="98bcd-141">Right-click the **Models** folder and select **Add**, then **Class...**. Name the class `NavMenuItem` and select **Add**.</span></span>

1. <span data-ttu-id="98bcd-142">Abra el archivo **NavMenuItem.CS** y reemplace el contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="98bcd-142">Open the **NavMenuItem.cs** file and replace its contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/NavMenuItem.cs" id="NavMenuItemSnippet":::

1. <span data-ttu-id="98bcd-143">Haga clic con el botón derecho en el proyecto **GraphTutorial** y seleccione **Agregar**y, a continuación, **nuevo elemento.**... Elija **Página de contenido** y asigne un `MenuPage`nombre a la página.</span><span class="sxs-lookup"><span data-stu-id="98bcd-143">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `MenuPage`.</span></span> <span data-ttu-id="98bcd-144">Elija **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="98bcd-144">Select **Add**.</span></span>

1. <span data-ttu-id="98bcd-145">Abra el archivo **MenuPage. Xaml** y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="98bcd-145">Open the **MenuPage.xaml** file and replace its contents with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/MenuPage.xaml":::

1. <span data-ttu-id="98bcd-146">Expanda **MenuPage. Xaml** en el **Explorador de soluciones** y abra el archivo **MenuPage.Xaml.CS** .</span><span class="sxs-lookup"><span data-stu-id="98bcd-146">Expand **MenuPage.xaml** in **Solution Explorer** and open the **MenuPage.xaml.cs** file.</span></span> <span data-ttu-id="98bcd-147">Reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="98bcd-147">Replace its contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/MenuPage.xaml.cs" id="MenuPageSnippet":::

    > [!NOTE]
    > <span data-ttu-id="98bcd-148">Visual Studio generará un informe de errores en **MenuPage.Xaml.CS**.</span><span class="sxs-lookup"><span data-stu-id="98bcd-148">Visual Studio will report errors in **MenuPage.xaml.cs**.</span></span> <span data-ttu-id="98bcd-149">Estos errores se resolverán en un paso posterior.</span><span class="sxs-lookup"><span data-stu-id="98bcd-149">These errors will be resolved in a later step.</span></span>

#### <a name="implement-the-welcome-page"></a><span data-ttu-id="98bcd-150">Implementar la página de bienvenida</span><span class="sxs-lookup"><span data-stu-id="98bcd-150">Implement the welcome page</span></span>

1. <span data-ttu-id="98bcd-151">Haga clic con el botón derecho en el proyecto **GraphTutorial** y seleccione **Agregar**y, a continuación, **nuevo elemento.**... Elija **Página de contenido** y asigne un `WelcomePage`nombre a la página.</span><span class="sxs-lookup"><span data-stu-id="98bcd-151">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `WelcomePage`.</span></span> <span data-ttu-id="98bcd-152">Elija **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="98bcd-152">Select **Add**.</span></span> <span data-ttu-id="98bcd-153">Abra el archivo **WelcomePage. Xaml** y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="98bcd-153">Open the **WelcomePage.xaml** file and replace its contents with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/WelcomePage.xaml":::

1. <span data-ttu-id="98bcd-154">Expanda **WelcomePage. Xaml** en el **Explorador de soluciones** y abra el archivo **WelcomePage.Xaml.CS** .</span><span class="sxs-lookup"><span data-stu-id="98bcd-154">Expand **WelcomePage.xaml** in **Solution Explorer** and open the **WelcomePage.xaml.cs** file.</span></span> <span data-ttu-id="98bcd-155">Agregue la siguiente función a la clase `WelcomePage`.</span><span class="sxs-lookup"><span data-stu-id="98bcd-155">Add the following function to the `WelcomePage` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/WelcomePage.xaml.cs" id="OnSignInSnippet":::

#### <a name="add-calendar-page"></a><span data-ttu-id="98bcd-156">Agregar página de calendario</span><span class="sxs-lookup"><span data-stu-id="98bcd-156">Add calendar page</span></span>

<span data-ttu-id="98bcd-157">Ahora, agregue una página de calendario.</span><span class="sxs-lookup"><span data-stu-id="98bcd-157">Now add a calendar page.</span></span> <span data-ttu-id="98bcd-158">Esto será simplemente un marcador de posición por ahora.</span><span class="sxs-lookup"><span data-stu-id="98bcd-158">This will just be a placeholder for now.</span></span>

1. <span data-ttu-id="98bcd-159">Haga clic con el botón derecho en el proyecto **GraphTutorial** y seleccione **Agregar**y, a continuación, **nuevo elemento.**... Elija **Página de contenido** y asigne un `CalendarPage`nombre a la página.</span><span class="sxs-lookup"><span data-stu-id="98bcd-159">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `CalendarPage`.</span></span> <span data-ttu-id="98bcd-160">Elija **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="98bcd-160">Select **Add**.</span></span>

#### <a name="update-mainpage-code-behind"></a><span data-ttu-id="98bcd-161">Actualizar código subyacente de la MainPage</span><span class="sxs-lookup"><span data-stu-id="98bcd-161">Update MainPage code-behind</span></span>

<span data-ttu-id="98bcd-162">Ahora que todas las páginas están en su ubicación, actualiza el código subyacente para **mainpage. Xaml**.</span><span class="sxs-lookup"><span data-stu-id="98bcd-162">Now that all of the pages are in place, update the code-behind for **MainPage.xaml**.</span></span>

1. <span data-ttu-id="98bcd-163">Expanda **mainpage. Xaml** en el **Explorador de soluciones** y abra el archivo **mainpage.Xaml.CS** y reemplace todo el contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="98bcd-163">Expand **MainPage.xaml** in **Solution Explorer** and open the **MainPage.xaml.cs** file and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/MainPage.xaml.cs" id="MainPageSnippet":::

1. <span data-ttu-id="98bcd-164">Guarde todos los cambios.</span><span class="sxs-lookup"><span data-stu-id="98bcd-164">Save all of your changes.</span></span> <span data-ttu-id="98bcd-165">Haz clic con el botón derecho en el proyecto que quieras ejecutar (Android, iOS o UWP) y selecciona **establecer como proyecto de inicio**.</span><span class="sxs-lookup"><span data-stu-id="98bcd-165">Right-click the project that you want to run (Android, iOS, or UWP) and select **Set as StartUp Project**.</span></span> <span data-ttu-id="98bcd-166">Presione **F5** o seleccione **depurar > iniciar depuración** en Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="98bcd-166">Press **F5** or select **Debug > Start Debugging** in Visual Studio.</span></span>

    ![Capturas de pantallas de las versiones de Android, iOS y UWP de la aplicación](./images/welcome-page.png)
