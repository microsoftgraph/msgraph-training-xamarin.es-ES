<!-- markdownlint-disable MD002 MD041 -->

Abra Visual Studio y seleccione **crear un nuevo proyecto**. En el cuadro de diálogo **crear un nuevo proyecto** , elija **aplicación móvil (Xamarin. Forms)** y, a continuación, elija **siguiente**.

![Cuadro de diálogo Crear nuevo proyecto de Visual Studio 2019](images/new-project-dialog.png)

En el cuadro de diálogo **configurar un nuevo proyecto** `GraphTutorial` , escriba para el nombre del **proyecto** y el nombre de la **solución**y, después, elija **crear**.

> [!IMPORTANT]
> Asegúrese de que escribe exactamente el mismo nombre para el proyecto de Visual Studio que se especifica en estas instrucciones de la práctica. El nombre del proyecto de Visual Studio se convierte en parte del espacio de nombres en el código. El código incluido en estas instrucciones depende del espacio de nombres que coincida con el nombre de proyecto de Visual Studio especificado en estas instrucciones. Si usa un nombre de proyecto diferente, el código no se compilará a menos que ajuste todos los espacios de nombres para que se correspondan con el nombre del proyecto de Visual Studio que ha especificado al crear el proyecto.

![Cuadro de diálogo Configurar nuevo proyecto de Visual Studio 2019](images/configure-new-project-dialog.png)

En el cuadro de diálogo **nueva aplicación para varias plataformas** , seleccione la plantilla **en blanco** y, a continuación, seleccione las plataformas que desea crear en **plataformas**. Seleccione **Aceptar** para crear la solución.

![Visual Studio 2019 cuadro de diálogo Nueva aplicación para varias plataformas](images/new-cross-platform-app-dialog.png)

Antes de continuar, instale algunos paquetes NuGet adicionales que usará más adelante.

- [Microsoft. Identity. Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) para administrar la autenticación y la administración de tokens de Azure ad.
- [Microsoft. Graph](https://www.nuget.org/packages/Microsoft.Graph/) para realizar llamadas a Microsoft Graph.

Seleccione **herramientas > el administrador de paquetes de NuGet > la consola del administrador de paquetes**. En la consola del administrador de paquetes, escriba los siguientes comandos.

```Powershell
Install-Package Microsoft.Identity.Client -Version 3.0.8 -Project GraphTutorial
Install-Package Microsoft.Identity.Client -Version 3.0.8 -Project GraphTutorial.Android
Install-Package Microsoft.Identity.Client -Version 3.0.8 -Project GraphTutorial.iOS
Install-Package Microsoft.Graph -Version 1.15.0 -Project GraphTutorial
```

## <a name="design-the-app"></a>Diseñar la aplicación

Para empezar, actualice `App` la clase para agregar variables para realizar un seguimiento del estado de autenticación y del usuario que ha iniciado sesión. En el **Explorador de soluciones**, expanda el proyecto **GraphTutorial** y, a continuación, expanda el archivo **app. Xaml** . Abra el archivo **app.Xaml.CS** y agregue las siguientes `using` instrucciones en la parte superior del archivo.

```cs
using System.ComponentModel;
using System.IO;
using System.Reflection;
using System.Threading.Tasks;
```

A continuación, agregue `INotifyPropertyChanged` la interfaz a la declaración de clase.

```cs
public partial class App : Application, INotifyPropertyChanged
```

Ahora, agregue las siguientes propiedades a `App` la clase.

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

Ahora, agregue las siguientes funciones a `App` la clase. Las `SignIn`funciones `SignOut`, y `GetUserInfo` son solo marcadores de posición por ahora.

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

La `GetUserPhoto` función devuelve una foto predeterminada por ahora. Puede proporcionar su propio archivo aquí, o puede descargar el que se usa en la muestra desde [GitHub](https://github.com/microsoftgraph/msgraph-training-xamarin/blob/master/tutorial/images/no-profile-pic.png). Copie el archivo en el `./GraphTutorial/GraphTutorial` directorio. Haga clic con el botón secundario en el proyecto **GraphTutorial** en el **Explorador de soluciones** y elija **Agregar**, a continuación, **elemento existente.**... Seleccione el `no-profile-pic.png` archivo y haga clic en **Agregar**. Ahora, haga clic con el botón secundario en el archivo en el **Explorador de soluciones** y elija **propiedades**. En la ventana **propiedades** , cambie el valor de la **acción de generación** a **recurso incrustado**.

![Captura de pantalla de la ventana Propiedades para el archivo PNG](./images/png-file-properties.png)

### <a name="app-navigation"></a>Navegación por la aplicación

A continuación, cambie la Página principal de la aplicación a una [Página principal-detalle](/xamarin/xamarin-forms/app-fundamentals/navigation/master-detail-page). Se proporcionará un menú de navegación para cambiar entre las vistas de la aplicación.

Abra el archivo **mainpage. Xaml** en el proyecto **GraphTutorial** y reemplace su contenido por lo siguiente.

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

#### <a name="implement-the-menu"></a>Implementar el menú

Empiece por crear un modelo para representar los elementos de menú. Haga clic con el botón derecho en el proyecto **GraphTutorial** , elija **Agregar**y, a continuación, **nueva carpeta**. Asigne un nombre `Models`a la carpeta.

Haga clic con el botón secundario en la carpeta **modelos** y elija **Agregar**y, a continuación, **clase...**. Asigne un nombre `NavMenuItem` a la clase y elija **Agregar**. Abra el archivo **NavMenuItem.CS** y reemplace el contenido por lo siguiente.

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

Ahora, agregue la página de menú. Haga clic con el botón derecho en el proyecto **GraphTutorial** y seleccione **Agregar**y, a continuación, **nuevo elemento.**... Elija **Página de contenido** y asigne un `MenuPage`nombre a la página. Seleccione **Agregar**. Abra el archivo **MenuPage. Xaml** y reemplace su contenido por lo siguiente.

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

Ahora, expanda **MenuPage. Xaml** en el **Explorador de soluciones** y abra el archivo **MenuPage.Xaml.CS** . Reemplace su contenido por lo siguiente.

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
> Visual Studio generará un informe de errores en **MenuPage.Xaml.CS**. Estos errores se resolverán en un paso posterior.

#### <a name="implement-the-welcome-page"></a>Implementar la página de bienvenida

Haga clic con el botón derecho en el proyecto **GraphTutorial** y seleccione **Agregar**y, a continuación, **nuevo elemento.**... Elija **Página de contenido** y asigne un `WelcomePage`nombre a la página. Seleccione **Agregar**. Abra el archivo **WelcomePage. Xaml** y reemplace su contenido por lo siguiente.

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

Ahora, expanda **WelcomePage. Xaml** en el **Explorador de soluciones** y abra el archivo **WelcomePage.Xaml.CS** . Agregue la siguiente función a la clase `WelcomePage`.

```cs
private void OnSignIn(object sender, EventArgs e)
{
    (App.Current.MainPage as MasterDetailPage).IsPresented = true;
}
```

#### <a name="add-calendar-page"></a>Agregar página de calendario

Ahora, agregue una página de calendario. Esto será simplemente un marcador de posición por ahora. Haga clic con el botón derecho en el proyecto **GraphTutorial** y seleccione **Agregar**y, a continuación, **nuevo elemento.**... Elija **Página de contenido** y asigne un `CalendarPage`nombre a la página. Seleccione **Agregar**.

Deje la página agregada como está por ahora.

#### <a name="update-mainpage-code-behind"></a>Actualizar código subyacente de la MainPage

Ahora que todas las páginas están en su ubicación, actualiza el código subyacente para **mainpage. Xaml**. Expanda **mainpage. Xaml** en el **Explorador de soluciones** y abra el archivo **mainpage.Xaml.CS** y reemplace todo el contenido por lo siguiente.

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

Guarde todos los cambios. Haz clic con el botón derecho en el proyecto que quieras ejecutar (Android, iOS o UWP) y elige **establecer como proyecto de inicio**. Presione **F5** o seleccione **depurar _GT_ iniciar** depuración en Visual Studio.

![Capturas de pantallas de las versiones de Android, iOS y UWP de la aplicación](./images/welcome-page.png)