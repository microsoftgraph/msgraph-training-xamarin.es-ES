<!-- markdownlint-disable MD002 MD041 -->

En esta sección agregará la capacidad de crear eventos en el calendario del usuario.

1. Agregue una nueva página para la nueva vista de eventos. Haga clic con el botón secundario **en el proyecto GraphTutorial** en el Explorador de soluciones y seleccione **Agregar > nuevo elemento...**. Elija **Página en** blanco, escriba en el `NewEventPage.xaml` **campo** Nombre y seleccione **Agregar**.

1. Abre **NewEventPage.xaml** y reemplaza su contenido por lo siguiente.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/NewEventPage.xaml" id="NewEventPageXamlSnippet":::

1. Abra **NewEventPage.xaml.cs** y agregue las `using` siguientes instrucciones en la parte superior del archivo.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/NewEventPage.xaml.cs" id="UsingStatementsSnippet":::

1. Agregue la **interfaz INotifyPropertyChange** a la **clase NewEventPage.** Reemplace la declaración de clase existente por lo siguiente.

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

1. Agregue las siguientes propiedades a la **clase NewEventPage.**

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/NewEventPage.xaml.cs" id="PropertiesSnippet":::

1. Agregue el siguiente código para crear el evento.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/NewEventPage.xaml.cs" id="CreateEventSnippet":::

1. Guarde los cambios y ejecute la aplicación. Inicie sesión, seleccione el **elemento de** menú Nuevo  evento, rellene el formulario y seleccione Crear para agregar un evento al calendario del usuario.

    ![Captura de pantalla de la nueva página de eventos](images/new-event-page.png)
