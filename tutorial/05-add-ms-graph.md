<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio incorporará Microsoft Graph en la aplicación. Para esta aplicación, usará la biblioteca cliente de [Microsoft Graph para .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) para realizar llamadas a Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Obtener eventos de calendario de Outlook

1. Abra **CalendarPage.xaml** en el **proyecto GraphTutorial** y reemplace su contenido por lo siguiente.

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

1. Abra **CalendarPage.xaml.cs** y agregue las `using` siguientes instrucciones en la parte superior del archivo.

    ```csharp
    using Microsoft.Graph;
    using Newtonsoft.Json;
    using System.Collections.ObjectModel;
    using System.ComponentModel;
    ```

1. Agregue la siguiente función a la `CalendarPage` clase para obtener el inicio de la semana actual en la zona horaria del usuario.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/CalendarPage.xaml.cs" id="GetStartOfWeekSnippet":::

1. Agregue la siguiente función a la clase para obtener los eventos del usuario `CalendarPage` y mostrar la respuesta JSON.

    ```csharp
    protected override async void OnAppearing()
    {
        base.OnAppearing();

        // Get start and end of week in user's time zone
        var startOfWeek = GetUtcStartOfWeekInTimeZone(DateTime.Today, App.UserTimeZone);
        var endOfWeek = startOfWeek.AddDays(7);

        var queryOptions = new List<QueryOption>
        {
            new QueryOption("startDateTime", startOfWeek.ToString("o")),
            new QueryOption("endDateTime", endOfWeek.ToString("o"))
        };

        // Get the events
        var events = await App.GraphClient.Me.CalendarView.Request(queryOptions)
            .Header("Prefer", $"outlook.timezone=\"{App.UserTimeZone.DisplayName}\"")
            .Select(e => new
            {
                e.Subject,
                e.Organizer,
                e.Start,
                e.End
            })
            .OrderBy("start/DateTime")
            .Top(50)
            .GetAsync();

        // Temporary
        JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
    }
    ```

    Ten en cuenta lo que hace `OnAppearing` el código.

    - La dirección URL a la que se llamará es `/v1.0/me/calendarview` .
        - Los `startDateTime` parámetros y definen el inicio y el final de la vista de `endDateTime` calendario.
        - El `Prefer: outlook.timezone` encabezado hace que el y de los eventos se devuelvan en la zona horaria del `start` `end` usuario.
        - La `Select` función limita los campos devueltos para cada evento a solo aquellos que la aplicación usará realmente.
        - La `OrderBy` función ordena los resultados por fecha y hora de inicio.
        - La `Top` función solicita como máximo 50 eventos.

1. Ejecuta la aplicación, inicia sesión y haz clic en el **elemento de** navegación Calendario del menú. Debería ver un volcado JSON de los eventos en el calendario del usuario.

## <a name="display-the-results"></a>Mostrar los resultados

Ahora puede reemplazar el volcado json por algo para mostrar los resultados de forma fácil de usar.

Empiece por crear un convertidor de valores [de](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) enlace para convertir los valores [dateTimeTimeZone](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) devueltos por Microsoft Graph en los formatos de fecha y hora que espera el usuario.

1. Haga clic con el **botón** secundario en la carpeta Modelos en el proyecto **GraphTutorial** y seleccione **Agregar**, después **Clase...**. Asigne un nombre a `GraphDateTimeTimeZoneConverter` la clase y seleccione **Agregar**.

1. Reemplace todo el contenido del archivo por lo siguiente.

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/GraphDateTimeTimeZoneConverter.cs" id="DateTimeConverterSnippet":::

1. Reemplaza todo el contenido de **CalendarPage.xaml** por lo siguiente.

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/CalendarPage.xaml" id="CalendarPageXamlSnippet":::

    Esto reemplaza el `Editor` archivo con un archivo `ListView` . El `DataTemplate` usado para representar cada elemento usa el para convertir las propiedades y del evento en una `GraphDateTimeTimeZoneConverter` `Start` `End` cadena.

1. Abra **CalendarPage.xaml.cs** y quite las siguientes líneas de la `OnAppearing` función.

    ```csharp
    // Temporary
    JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
    ```

1. En su lugar, agregue el siguiente código.

    ```csharp
    // Add the events to the list view
    CalendarList.ItemsSource = events.CurrentPage.ToList();
    ```

1. Ejecuta la aplicación, inicia sesión y haz clic en el **elemento de** navegación Calendario. Debería ver la lista de eventos con los valores **de Inicio** **y** Fin con formato.

    ![Captura de pantalla de la tabla de eventos](./images/calendar-page.png)
