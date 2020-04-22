<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, incorporará Microsoft Graph a la aplicación. Para esta aplicación, usará la [biblioteca de cliente de Microsoft Graph para .net](https://github.com/microsoftgraph/msgraph-sdk-dotnet) para realizar llamadas a Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Obtener eventos de calendario de Outlook

1. Abra **CalendarPage. Xaml** en el proyecto **GraphTutorial** y reemplace su contenido por lo siguiente.

    ```xaml
    <?xml version="1.0" encoding="utf-8" ?>
    <ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
                 xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
                 Title="Calendar"
                 x:Class="GraphTutorial.CalendarPage">
        <ContentPage.Content>
            <StackLayout>
                <Editor x:Name="JSONResponse"
                        IsSpellCheckEnabled="False"
                        HorizontalOptions="FillAndExpand"
                        VerticalOptions="FillAndExpand"/>
            </StackLayout>
        </ContentPage.Content>
    </ContentPage>
    ```

1. Abra **CalendarPage.Xaml.CS** y agregue las siguientes `using` instrucciones en la parte superior del archivo.

    ```csharp
    using Microsoft.Graph;
    using Newtonsoft.Json;
    using System.Collections.ObjectModel;
    using System.ComponentModel;
    ```

1. Agregue la siguiente función a la `CalendarPage` clase para obtener los eventos del usuario y mostrar la respuesta de JSON.

    ```csharp
    protected override async void OnAppearing()
    {
        base.OnAppearing();

        // Get the events
        var events = await App.GraphClient.Me.Events.Request()
            .Select(e => new
            {
                e.Subject,
                e.Organizer,
                e.Start,
                e.End
            })
            .OrderBy("createdDateTime DESC")
            .GetAsync();

        // Temporary
        JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
    }
    ```

    Tenga en cuenta lo que `OnAppearing` hace el código.

    - La dirección URL a la que se `/v1.0/me/events`llamará es.
    - La `Select` función limita los campos devueltos para cada evento a solo aquellos que la vista usará realmente.
    - La `OrderBy` función ordena los resultados por la fecha y hora en que se crearon, con el elemento más reciente en primer lugar.

1. Ejecute la aplicación, inicie sesión y haga clic en el elemento de navegación **calendario** en el menú. Debería ver un volcado JSON de los eventos en el calendario del usuario.

## <a name="display-the-results"></a>Mostrar los resultados

Ahora puede reemplazar el volcado de JSON con algo para mostrar los resultados de forma fácil de uso.

Para empezar, cree un [convertidor de valores de enlace](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) para convertir los valores de [dateTimeTimeZone](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) devueltos por Microsoft Graph en los formatos de fecha y hora que el usuario espera.

1. Haga clic con el botón derecho en la carpeta **modelos** en el proyecto **GraphTutorial** y seleccione **Agregar**y, a continuación, **clase...**. Asigne un nombre `GraphDateTimeTimeZoneConverter` a la clase y seleccione **Agregar**.

1. Reemplace todo el contenido del archivo por lo siguiente.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/GraphDateTimeTimeZoneConverter.cs" id="DateTimeConverterSnippet":::

1. Reemplace todo el contenido de **CalendarPage. Xaml** por lo siguiente.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/CalendarPage.xaml":::

    Esto reemplaza `Editor` por un `ListView`. El `DataTemplate` que se usa para representar cada elemento `GraphDateTimeTimeZoneConverter` utiliza el para `Start` convertir `End` las propiedades y del evento en una cadena.

1. Abra **CalendarPage.Xaml.CS** y quite las siguientes líneas de la `OnAppearing` función.

    ```csharp
    // Temporary
    JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
    ```

1. En su ubicación, agregue el siguiente código.

    ```csharp
    // Add the events to the list view
    CalendarList.ItemsSource = events.CurrentPage.ToList();
    ```

1. Ejecute la aplicación, inicie sesión y haga clic en el elemento de navegación **calendario** . Debe ver la lista de eventos con los valores de **Inicio** y **finalización** con formato.

    ![Captura de pantalla de la tabla de eventos](./images/calendar-page.png)
