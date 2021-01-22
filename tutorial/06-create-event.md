<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="d43a5-101">En esta sección agregará la capacidad de crear eventos en el calendario del usuario.</span><span class="sxs-lookup"><span data-stu-id="d43a5-101">In this section you will add the ability to create events on the user's calendar.</span></span>

1. <span data-ttu-id="d43a5-102">Agregue una nueva página para la nueva vista de eventos.</span><span class="sxs-lookup"><span data-stu-id="d43a5-102">Add a new page for the new event view.</span></span> <span data-ttu-id="d43a5-103">Haga clic con el botón secundario **en el proyecto GraphTutorial** en el Explorador de soluciones y seleccione **Agregar > nuevo elemento...**. Elija **Página en** blanco, escriba en el `NewEventPage.xaml` **campo** Nombre y seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="d43a5-103">Right-click the **GraphTutorial** project in Solution Explorer and select **Add > New Item...**. Choose **Blank Page**, enter `NewEventPage.xaml` in the **Name** field, and select **Add**.</span></span>

1. <span data-ttu-id="d43a5-104">Abre **NewEventPage.xaml** y reemplaza su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="d43a5-104">Open **NewEventPage.xaml** and replace its contents with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/NewEventPage.xaml" id="NewEventPageXamlSnippet":::

1. <span data-ttu-id="d43a5-105">Abra **NewEventPage.xaml.cs** y agregue las `using` siguientes instrucciones en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="d43a5-105">Open **NewEventPage.xaml.cs** and add the following `using` statements to the top of the file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/NewEventPage.xaml.cs" id="UsingStatementsSnippet":::

1. <span data-ttu-id="d43a5-106">Agregue la **interfaz INotifyPropertyChange** a la **clase NewEventPage.**</span><span class="sxs-lookup"><span data-stu-id="d43a5-106">Add the **INotifyPropertyChange** interface to the **NewEventPage** class.</span></span> <span data-ttu-id="d43a5-107">Reemplace la declaración de clase existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="d43a5-107">Replace the existing class declaration with the following.</span></span>

    ```csharp
    [XamlCompilation(XamlCompilationOptions.Compile)]
    public partial class NewEventPage : ContentPage, INotifyPropertyChanged
    {
        public NewEventPage()
        {
            InitializeComponent();
            BindingContext = this;
        }
    }
    ```

1. <span data-ttu-id="d43a5-108">Agregue las siguientes propiedades a la **clase NewEventPage.**</span><span class="sxs-lookup"><span data-stu-id="d43a5-108">Add the following properties to the **NewEventPage** class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/NewEventPage.xaml.cs" id="PropertiesSnippet":::

1. <span data-ttu-id="d43a5-109">Agregue el siguiente código para crear el evento.</span><span class="sxs-lookup"><span data-stu-id="d43a5-109">Add the following code to create the event.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/NewEventPage.xaml.cs" id="CreateEventSnippet":::

1. <span data-ttu-id="d43a5-110">Guarde los cambios y ejecute la aplicación.</span><span class="sxs-lookup"><span data-stu-id="d43a5-110">Save your changes and run the app.</span></span> <span data-ttu-id="d43a5-111">Inicie sesión, seleccione el **elemento de** menú Nuevo  evento, rellene el formulario y seleccione Crear para agregar un evento al calendario del usuario.</span><span class="sxs-lookup"><span data-stu-id="d43a5-111">Sign in, select the **New event** menu item, fill in the form, and select **Create** to add an event to the user's calendar.</span></span>

    ![Captura de pantalla de la nueva página de eventos](images/new-event-page.png)
