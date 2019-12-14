<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f2a1a-101">Abra Visual Studio y seleccione **crear un nuevo proyecto**.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-101">Open Visual Studio, and select **Create a new project**.</span></span> <span data-ttu-id="f2a1a-102">En el cuadro de diálogo **crear un nuevo proyecto** , elija **aplicación móvil (Xamarin. Forms)** y, a continuación, seleccione **siguiente**.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-102">In the **Create a new project** dialog, choose **Mobile App (Xamarin.Forms)**, then select **Next**.</span></span>

![Cuadro de diálogo Crear nuevo proyecto de Visual Studio 2019](images/new-project-dialog.png)

<span data-ttu-id="f2a1a-104">En el cuadro de diálogo **configurar un nuevo proyecto** `GraphTutorial` , escriba para el nombre del **proyecto** y el nombre de la **solución**y, después, seleccione **crear**.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-104">In the **Configure a new project** dialog, enter `GraphTutorial` for the **Project name** and **Solution name**, then select **Create**.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="f2a1a-105">Asegúrese de que escribe exactamente el mismo nombre para el proyecto de Visual Studio que se especifica en estas instrucciones de la práctica.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-105">Ensure that you enter the exact same name for the Visual Studio Project that is specified in these lab instructions.</span></span> <span data-ttu-id="f2a1a-106">El nombre del proyecto de Visual Studio se convierte en parte del espacio de nombres en el código.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-106">The Visual Studio Project name becomes part of the namespace in the code.</span></span> <span data-ttu-id="f2a1a-107">El código incluido en estas instrucciones depende del espacio de nombres que coincida con el nombre de proyecto de Visual Studio especificado en estas instrucciones.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-107">The code inside these instructions depends on the namespace matching the Visual Studio Project name specified in these instructions.</span></span> <span data-ttu-id="f2a1a-108">Si usa un nombre de proyecto diferente, el código no se compilará a menos que ajuste todos los espacios de nombres para que se correspondan con el nombre del proyecto de Visual Studio que ha especificado al crear el proyecto.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-108">If you use a different project name the code will not compile unless you adjust all the namespaces to match the Visual Studio Project name you enter when you create the project.</span></span>

![Cuadro de diálogo Configurar nuevo proyecto de Visual Studio 2019](images/configure-new-project-dialog.png)

<span data-ttu-id="f2a1a-110">En el cuadro de diálogo **nueva aplicación para varias plataformas** , seleccione la plantilla **en blanco** y, a continuación, seleccione las plataformas que desea crear en **plataformas**.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-110">In the **New Cross Platform App** dialog, select the **Blank** template, and select the platforms you want to build under **Platforms**.</span></span> <span data-ttu-id="f2a1a-111">Seleccione **Aceptar** para crear la solución.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-111">Select **OK** to create the solution.</span></span>

![Visual Studio 2019 cuadro de diálogo Nueva aplicación para varias plataformas](images/new-cross-platform-app-dialog.png)

<span data-ttu-id="f2a1a-113">Antes de continuar, instale algunos paquetes NuGet adicionales que usará más adelante.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-113">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="f2a1a-114">[Microsoft. Identity. Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) para administrar la autenticación y la administración de tokens de Azure ad.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-114">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) to handle Azure AD authentication and token management.</span></span>
- <span data-ttu-id="f2a1a-115">[Microsoft. Graph](https://www.nuget.org/packages/Microsoft.Graph/) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-115">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) for making calls to the Microsoft Graph.</span></span>

<span data-ttu-id="f2a1a-116">Seleccione **herramientas > el administrador de paquetes de NuGet > consola del administrador de paquetes**.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-116">Select **Tools > NuGet Package Manager > Package Manager Console**.</span></span> <span data-ttu-id="f2a1a-117">En la consola del administrador de paquetes, escriba los siguientes comandos.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-117">In the Package Manager Console, enter the following commands.</span></span>

```Powershell
Install-Package Microsoft.Identity.Client -Version 4.7.1 -Project GraphTutorial
Install-Package Microsoft.Identity.Client -Version 4.7.1 -Project GraphTutorial.Android
Install-Package Microsoft.Identity.Client -Version 4.7.1 -Project GraphTutorial.iOS
Install-Package Microsoft.Graph -Version 1.20.0 -Project GraphTutorial
```

## <a name="design-the-app"></a><span data-ttu-id="f2a1a-118">Diseñar la aplicación</span><span class="sxs-lookup"><span data-stu-id="f2a1a-118">Design the app</span></span>

<span data-ttu-id="f2a1a-119">Para empezar, actualice `App` la clase para agregar variables para realizar un seguimiento del estado de autenticación y del usuario que ha iniciado sesión.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-119">Start by updating the `App` class to add variables to track the authentication state and the signed-in user.</span></span> <span data-ttu-id="f2a1a-120">En el **Explorador de soluciones**, expanda el proyecto **GraphTutorial** y, a continuación, expanda el archivo **app. Xaml** .</span><span class="sxs-lookup"><span data-stu-id="f2a1a-120">In **Solution Explorer**, expand the **GraphTutorial** project, then expand the **App.xaml** file.</span></span> <span data-ttu-id="f2a1a-121">Abra el archivo **app.Xaml.CS** y agregue las siguientes `using` instrucciones en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-121">Open the **App.xaml.cs** file and add the following `using` statements to the top of the file.</span></span>

```cs
using System.ComponentModel;
using System.IO;
using System.Reflection;
using System.Threading.Tasks;
```

<span data-ttu-id="f2a1a-122">A continuación, agregue `INotifyPropertyChanged` la interfaz a la declaración de clase.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-122">Next, add the `INotifyPropertyChanged` interface to the class declaration.</span></span>

```cs
public partial class App : Application, INotifyPropertyChanged
```

<span data-ttu-id="f2a1a-123">Ahora, agregue las siguientes propiedades a `App` la clase.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-123">Now add the following properties to the `App` class.</span></span>

```cs
// Is a user signed in?
private bool isSignedIn;
public bool IsSignedIn
{
    get { return isSignedIn; }
    set
    {
        isSignedIn = value;
        OnPropertyChanged("IsSignedIn");
        OnPropertyChanged("IsSignedOut");
    }
}

public bool IsSignedOut { get { return !isSignedIn; } }

// The user's display name
private string userName;
public string UserName
{
    get { return userName; }
    set
    {
        userName = value;
        OnPropertyChanged("UserName");
    }
}

// The user's email address
private string userEmail;
public string UserEmail
{
    get { return userEmail; }
    set
    {
        userEmail = value;
        OnPropertyChanged("UserEmail");
    }
}

// The user's profile photo
private ImageSource userPhoto;
public ImageSource UserPhoto
{
    get { return userPhoto; }
    set
    {
        userPhoto = value;
        OnPropertyChanged("UserPhoto");
    }
}
```

<span data-ttu-id="f2a1a-124">Ahora, agregue las siguientes funciones a `App` la clase.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-124">Now add the following functions to the `App` class.</span></span> <span data-ttu-id="f2a1a-125">Las `SignIn`funciones `SignOut`, y `GetUserInfo` son solo marcadores de posición por ahora.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-125">The `SignIn`, `SignOut`, and `GetUserInfo` functions are just placeholders for now.</span></span>

```cs
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

<span data-ttu-id="f2a1a-126">La `GetUserPhoto` función devuelve una foto predeterminada por ahora.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-126">The `GetUserPhoto` function returns a default photo for now.</span></span> <span data-ttu-id="f2a1a-127">Puede proporcionar su propio archivo aquí, o puede descargar el que se usa en la muestra desde [GitHub](https://github.com/microsoftgraph/msgraph-training-xamarin/blob/master/tutorial/images/no-profile-pic.png).</span><span class="sxs-lookup"><span data-stu-id="f2a1a-127">You can either supply your own file here, or you can download the one used in the sample from [GitHub](https://github.com/microsoftgraph/msgraph-training-xamarin/blob/master/tutorial/images/no-profile-pic.png).</span></span> <span data-ttu-id="f2a1a-128">Copie el archivo en el `./GraphTutorial/GraphTutorial` directorio.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-128">Copy the file to the `./GraphTutorial/GraphTutorial` directory.</span></span> <span data-ttu-id="f2a1a-129">Haga clic con el botón derecho en el proyecto **GraphTutorial** en el **Explorador de soluciones** y seleccione **Agregar**, a continuación, **elemento existente...**. Seleccione el `no-profile-pic.png` archivo y haga clic en **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-129">Right-click the **GraphTutorial** project in **Solution Explorer** and select **Add**, then **Existing Item...**. Select the `no-profile-pic.png` file and select **Add**.</span></span> <span data-ttu-id="f2a1a-130">Ahora, haga clic con el botón derecho en el archivo en el **Explorador de soluciones** y seleccione **propiedades**.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-130">Now right-click the file in **Solution Explorer** and select **Properties**.</span></span> <span data-ttu-id="f2a1a-131">En la ventana **propiedades** , cambie el valor de la **acción de generación** a **recurso incrustado**.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-131">In the **Properties** window, change the value of **Build Action** to **Embedded resource**.</span></span>

![Captura de pantalla de la ventana Propiedades para el archivo PNG](./images/png-file-properties.png)

### <a name="app-navigation"></a><span data-ttu-id="f2a1a-133">Navegación por la aplicación</span><span class="sxs-lookup"><span data-stu-id="f2a1a-133">App navigation</span></span>

<span data-ttu-id="f2a1a-134">A continuación, cambie la Página principal de la aplicación a una [Página principal-detalle](/xamarin/xamarin-forms/app-fundamentals/navigation/master-detail-page).</span><span class="sxs-lookup"><span data-stu-id="f2a1a-134">Next, change the application's main page to a [Master-Detail page](/xamarin/xamarin-forms/app-fundamentals/navigation/master-detail-page).</span></span> <span data-ttu-id="f2a1a-135">Se proporcionará un menú de navegación para cambiar entre las vistas de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-135">This will provide a navigation menu to switch between view in the app.</span></span>

<span data-ttu-id="f2a1a-136">Abra el archivo **mainpage. Xaml** en el proyecto **GraphTutorial** y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-136">Open the **MainPage.xaml** file in the **GraphTutorial** project and replace its contents with the following.</span></span>

```xml
<?xml version="1.0" encoding="utf-8" ?>
<MasterDetailPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:GraphTutorial"
             x:Class="GraphTutorial.MainPage">

    <MasterDetailPage.Master>
        <local:MenuPage/>
    </MasterDetailPage.Master>

    <MasterDetailPage.Detail>
        <NavigationPage>
            <x:Arguments>
                <local:WelcomePage/>
            </x:Arguments>
        </NavigationPage>
    </MasterDetailPage.Detail>

</MasterDetailPage>
```

#### <a name="implement-the-menu"></a><span data-ttu-id="f2a1a-137">Implementar el menú</span><span class="sxs-lookup"><span data-stu-id="f2a1a-137">Implement the menu</span></span>

<span data-ttu-id="f2a1a-138">Empiece por crear un modelo para representar los elementos de menú.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-138">Start by creating a model to represent the menu items.</span></span> <span data-ttu-id="f2a1a-139">Haga clic con el botón derecho en el proyecto **GraphTutorial** y seleccione **Agregar**y, a continuación, **nueva carpeta**.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-139">Right-click the **GraphTutorial** project and select **Add**, then **New Folder**.</span></span> <span data-ttu-id="f2a1a-140">Asigne un nombre `Models`a la carpeta.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-140">Name the folder `Models`.</span></span>

<span data-ttu-id="f2a1a-141">Haga clic con el botón secundario en la carpeta **modelos** y seleccione **Agregar**y, a continuación, **clase...**. Asigne un nombre `NavMenuItem` a la clase y seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-141">Right-click the **Models** folder and select **Add**, then **Class...**. Name the class `NavMenuItem` and select **Add**.</span></span> <span data-ttu-id="f2a1a-142">Abra el archivo **NavMenuItem.CS** y reemplace el contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-142">Open the **NavMenuItem.cs** file and replace its contents with the following.</span></span>

```cs
namespace GraphTutorial.Models
{
    public enum MenuItemType
    {
        Welcome,
        Calendar
    }

    public class NavMenuItem
    {
        public MenuItemType Id { get; set; }

        public string Title { get; set; }
    }
}
```

<span data-ttu-id="f2a1a-143">Ahora, agregue la página de menú.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-143">Now add the menu page.</span></span> <span data-ttu-id="f2a1a-144">Haga clic con el botón derecho en el proyecto **GraphTutorial** y seleccione **Agregar**y, a continuación, **nuevo elemento.**... Elija **Página de contenido** y asigne un `MenuPage`nombre a la página.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-144">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `MenuPage`.</span></span> <span data-ttu-id="f2a1a-145">Seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-145">Select **Add**.</span></span> <span data-ttu-id="f2a1a-146">Abra el archivo **MenuPage. Xaml** y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-146">Open the **MenuPage.xaml** file and replace its contents with the following.</span></span>

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:ios="clr-namespace:Xamarin.Forms.PlatformConfiguration.iOSSpecific;assembly=Xamarin.Forms.Core"
             ios:Page.UseSafeArea="true"
             Title="Menu"
             x:Class="GraphTutorial.MenuPage">
    <ContentPage.Padding>
        <OnPlatform x:TypeArguments="Thickness">
            <On Platform="UWP" Value="10, 10, 10, 10" />
        </OnPlatform>
    </ContentPage.Padding>
    <ContentPage.Content>
        <StackLayout VerticalOptions="Start" HorizontalOptions="Center">
            <StackLayout x:Name="UserArea" />

            <!-- Signed out UI -->
            <StackLayout IsVisible="{Binding Path=IsSignedOut, Source={x:Static Application.Current}}">
                <Label Text="Sign in to get started"
                       HorizontalOptions="Center"
                       FontAttributes="Bold"
                       FontSize="Medium"
                       Margin="10,20,10,20" />
                <Button Text="Sign in"
                        Clicked="OnSignIn"
                        HorizontalOptions="Center" />
            </StackLayout>

            <!-- Signed in UI -->
            <StackLayout IsVisible="{Binding Path=IsSignedIn, Source={x:Static Application.Current}}">
                <Image Source="{Binding Path=UserPhoto, Source={x:Static Application.Current}}"
                       HorizontalOptions="Center"
                       Margin="0,20,0,10" />
                <Label Text="{Binding Path=UserName, Source={x:Static Application.Current}}"
                       HorizontalOptions="Center"
                       FontAttributes="Bold"
                       FontSize="Small" />
                <Label Text="{Binding Path=UserEmail, Source={x:Static Application.Current}}"
                       HorizontalOptions="Center"
                       FontAttributes="Italic" />
                <Button Text="Sign out"
                        Margin="0,20,0,20"
                        Clicked="OnSignOut"
                        HorizontalOptions="Center" />
                <ListView x:Name="ListViewMenu"
                          HasUnevenRows="True"
                          HorizontalOptions="Start">
                    <ListView.ItemTemplate>
                        <DataTemplate>
                            <ViewCell>
                                <Grid Padding="10">
                                    <Label Text="{Binding Title}" FontSize="20"/>
                                </Grid>
                            </ViewCell>
                        </DataTemplate>
                    </ListView.ItemTemplate>
                </ListView>
            </StackLayout>
        </StackLayout>
    </ContentPage.Content>
</ContentPage>
```

<span data-ttu-id="f2a1a-147">Ahora, expanda **MenuPage. Xaml** en el **Explorador de soluciones** y abra el archivo **MenuPage.Xaml.CS** .</span><span class="sxs-lookup"><span data-stu-id="f2a1a-147">Now, expand **MenuPage.xaml** in **Solution Explorer** and open the **MenuPage.xaml.cs** file.</span></span> <span data-ttu-id="f2a1a-148">Reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-148">Replace its contents with the following.</span></span>

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

using Xamarin.Forms;
using Xamarin.Forms.Xaml;

using GraphTutorial.Models;

namespace GraphTutorial
{
    [XamlCompilation(XamlCompilationOptions.Compile)]
    public partial class MenuPage : ContentPage
    {
        MainPage RootPage => Application.Current.MainPage as MainPage;
        List<NavMenuItem> menuItems;

        public MenuPage ()
        {
            InitializeComponent ();

            // Add items to the menu
            menuItems = new List<NavMenuItem>
            {
                new NavMenuItem {Id = MenuItemType.Welcome, Title="Home" },
                new NavMenuItem {Id = MenuItemType.Calendar, Title="Calendar" }
            };
            ListViewMenu.ItemsSource = menuItems;

            // Initialize the selected item
            ListViewMenu.SelectedItem = menuItems[0];

            // Handle the ItemSelected event to navigate to the
            // selected page
            ListViewMenu.ItemSelected += async (sender, e) =>
            {
                if (e.SelectedItem == null)
                    return;

                var id = (int)((NavMenuItem)e.SelectedItem).Id;
                await RootPage.NavigateFromMenu(id);
            };
        }

        private async void OnSignOut(object sender, EventArgs e)
        {
            var signout = await DisplayAlert("Sign out?", "Do you want to sign out?", "Yes", "No");
            if (signout)
            {
                await (Application.Current as App).SignOut();
            }
        }

        private async void OnSignIn(object sender, EventArgs e)
        {
            await (Application.Current as App).SignIn();
        }
    }
}
```

> [!NOTE]
> <span data-ttu-id="f2a1a-149">Visual Studio generará un informe de errores en **MenuPage.Xaml.CS**.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-149">Visual Studio will report errors in **MenuPage.xaml.cs**.</span></span> <span data-ttu-id="f2a1a-150">Estos errores se resolverán en un paso posterior.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-150">These errors will be resolved in a later step.</span></span>

#### <a name="implement-the-welcome-page"></a><span data-ttu-id="f2a1a-151">Implementar la página de bienvenida</span><span class="sxs-lookup"><span data-stu-id="f2a1a-151">Implement the welcome page</span></span>

<span data-ttu-id="f2a1a-152">Haga clic con el botón derecho en el proyecto **GraphTutorial** y seleccione **Agregar**y, a continuación, **nuevo elemento.**... Elija **Página de contenido** y asigne un `WelcomePage`nombre a la página.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-152">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `WelcomePage`.</span></span> <span data-ttu-id="f2a1a-153">Seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-153">Select **Add**.</span></span> <span data-ttu-id="f2a1a-154">Abra el archivo **WelcomePage. Xaml** y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-154">Open the **WelcomePage.xaml** file and replace its contents with the following.</span></span>

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             Title="Home"
             x:Class="GraphTutorial.WelcomePage">
    <ContentPage.Padding>
        <OnPlatform x:TypeArguments="Thickness">
            <On Platform="UWP" Value="10, 10, 10, 10" />
        </OnPlatform>
    </ContentPage.Padding>
    <ContentPage.Content>
        <StackLayout>
            <Label Text="Graph Xamarin Tutorial App"
                       HorizontalOptions="Center"
                       FontAttributes="Bold"
                       FontSize="Large"
                       Margin="10,20,10,20" />

            <!-- Signed out UI -->
            <StackLayout IsVisible="{Binding Path=IsSignedOut, Source={x:Static Application.Current}}">
                <Label Text="Please sign in to get started"
                       HorizontalOptions="Center"
                       FontSize="Medium"
                       Margin="10,0,10,20"/>
                <Button Text="Sign in"
                        HorizontalOptions="Center"
                        Clicked="OnSignIn" />
            </StackLayout>

            <!-- Signed in UI -->
            <StackLayout IsVisible="{Binding Path=IsSignedIn, Source={x:Static Application.Current}}">
                <Label Text="{Binding Path=UserName, Source={x:Static Application.Current}, StringFormat='Welcome \{0\}!'}"
                       HorizontalOptions="Center"
                       FontSize="Medium"/>
            </StackLayout>
        </StackLayout>
    </ContentPage.Content>
</ContentPage>
```

<span data-ttu-id="f2a1a-155">Ahora, expanda **WelcomePage. Xaml** en el **Explorador de soluciones** y abra el archivo **WelcomePage.Xaml.CS** .</span><span class="sxs-lookup"><span data-stu-id="f2a1a-155">Now, expand **WelcomePage.xaml** in **Solution Explorer** and open the **WelcomePage.xaml.cs** file.</span></span> <span data-ttu-id="f2a1a-156">Agregue la siguiente función a la clase `WelcomePage`.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-156">Add the following function to the `WelcomePage` class.</span></span>

```cs
private void OnSignIn(object sender, EventArgs e)
{
    (App.Current.MainPage as MasterDetailPage).IsPresented = true;
}
```

#### <a name="add-calendar-page"></a><span data-ttu-id="f2a1a-157">Agregar página de calendario</span><span class="sxs-lookup"><span data-stu-id="f2a1a-157">Add calendar page</span></span>

<span data-ttu-id="f2a1a-158">Ahora, agregue una página de calendario.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-158">Now add a calendar page.</span></span> <span data-ttu-id="f2a1a-159">Esto será simplemente un marcador de posición por ahora.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-159">This will just be a placeholder for now.</span></span> <span data-ttu-id="f2a1a-160">Haga clic con el botón derecho en el proyecto **GraphTutorial** y seleccione **Agregar**y, a continuación, **nuevo elemento.**... Elija **Página de contenido** y asigne un `CalendarPage`nombre a la página.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-160">Right-click the **GraphTutorial** project and select **Add**, then **New Item...**. Choose **Content Page** and name the page `CalendarPage`.</span></span> <span data-ttu-id="f2a1a-161">Seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-161">Select **Add**.</span></span>

<span data-ttu-id="f2a1a-162">Deje la página agregada como está por ahora.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-162">Leave the added page as-is for now.</span></span>

#### <a name="update-mainpage-code-behind"></a><span data-ttu-id="f2a1a-163">Actualizar código subyacente de la MainPage</span><span class="sxs-lookup"><span data-stu-id="f2a1a-163">Update MainPage code-behind</span></span>

<span data-ttu-id="f2a1a-164">Ahora que todas las páginas están en su ubicación, actualiza el código subyacente para **mainpage. Xaml**.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-164">Now that all of the pages are in place, update the code-behind for **MainPage.xaml**.</span></span> <span data-ttu-id="f2a1a-165">Expanda **mainpage. Xaml** en el **Explorador de soluciones** y abra el archivo **mainpage.Xaml.CS** y reemplace todo el contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-165">Expand **MainPage.xaml** in **Solution Explorer** and open the **MainPage.xaml.cs** file and replace its entire contents with the following.</span></span>

```cs
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Reflection;
using System.Text;
using System.Threading.Tasks;
using Xamarin.Forms;
using GraphTutorial.Models;

namespace GraphTutorial
{
    public partial class MainPage : MasterDetailPage
    {
        Dictionary<int, NavigationPage> MenuPages = new Dictionary<int, NavigationPage>();

        public MainPage()
        {
            InitializeComponent();

            MasterBehavior = MasterBehavior.Popover;

            // Load the welcome page at start
            MenuPages.Add((int)MenuItemType.Welcome, (NavigationPage)Detail);
        }

        // Navigate to the selected page
        public async Task NavigateFromMenu(int id)
        {
            if (!MenuPages.ContainsKey(id))
            {
                switch (id)
                {
                    case (int)MenuItemType.Welcome:
                        MenuPages.Add(id, new NavigationPage(new WelcomePage()));
                        break;
                    case (int)MenuItemType.Calendar:
                        MenuPages.Add(id, new NavigationPage(new CalendarPage()));
                        break;
                }
            }

            var newPage = MenuPages[id];

            if (newPage != null && Detail != newPage)
            {
                Detail = newPage;

                if (Device.RuntimePlatform == Device.Android)
                    await Task.Delay(100);

                IsPresented = false;
            }
        }
    }
}
```

<span data-ttu-id="f2a1a-166">Guarde todos los cambios.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-166">Save all of your changes.</span></span> <span data-ttu-id="f2a1a-167">Haz clic con el botón derecho en el proyecto que quieras ejecutar (Android, iOS o UWP) y selecciona **establecer como proyecto de inicio**.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-167">Right-click the project that you want to run (Android, iOS, or UWP) and select **Set as StartUp Project**.</span></span> <span data-ttu-id="f2a1a-168">Presione **F5** o seleccione **depurar > iniciar depuración** en Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="f2a1a-168">Press **F5** or select **Debug > Start Debugging** in Visual Studio.</span></span>

![Capturas de pantallas de las versiones de Android, iOS y UWP de la aplicación](./images/welcome-page.png)
