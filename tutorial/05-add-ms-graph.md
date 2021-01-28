<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="46291-101">En este ejercicio incorporará Microsoft Graph en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="46291-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="46291-102">Para esta aplicación, usará la biblioteca cliente de [Microsoft Graph para .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="46291-102">For this application, you will use the [Microsoft Graph Client Library for .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="46291-103">Obtener eventos de calendario de Outlook</span><span class="sxs-lookup"><span data-stu-id="46291-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="46291-104">Abra **CalendarPage.xaml** en el **proyecto GraphTutorial** y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="46291-104">Open **CalendarPage.xaml** in the **GraphTutorial** project and replace its contents with the following.</span></span>

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

1. <span data-ttu-id="46291-105">Abra **CalendarPage.xaml.cs** y agregue las `using` siguientes instrucciones en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="46291-105">Open **CalendarPage.xaml.cs** and add the following `using` statements at the top of the file.</span></span>

    ```csharp
    using Microsoft.Graph;
    using Newtonsoft.Json;
    using System.Collections.ObjectModel;
    using System.ComponentModel;
    ```

1. <span data-ttu-id="46291-106">Agregue la siguiente función a la `CalendarPage` clase para obtener el inicio de la semana actual en la zona horaria del usuario.</span><span class="sxs-lookup"><span data-stu-id="46291-106">Add the following function to the `CalendarPage` class to get the start of the current week in the user's time zone.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/CalendarPage.xaml.cs" id="GetStartOfWeekSnippet":::

1. <span data-ttu-id="46291-107">Agregue la siguiente función a la clase para obtener los eventos del usuario `CalendarPage` y mostrar la respuesta JSON.</span><span class="sxs-lookup"><span data-stu-id="46291-107">Add the following function to the `CalendarPage` class to get the user's events and display the JSON response.</span></span>

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

    <span data-ttu-id="46291-108">Ten en cuenta lo que hace `OnAppearing` el código.</span><span class="sxs-lookup"><span data-stu-id="46291-108">Consider what the code in `OnAppearing` is doing.</span></span>

    - <span data-ttu-id="46291-109">La dirección URL a la que se llamará es `/v1.0/me/calendarview` .</span><span class="sxs-lookup"><span data-stu-id="46291-109">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
        - <span data-ttu-id="46291-110">Los `startDateTime` parámetros y definen el inicio y el final de la vista de `endDateTime` calendario.</span><span class="sxs-lookup"><span data-stu-id="46291-110">The `startDateTime` and `endDateTime` parameters define the start and end of the calendar view.</span></span>
        - <span data-ttu-id="46291-111">El `Prefer: outlook.timezone` encabezado hace que el y de los eventos se devuelvan en la zona horaria del `start` `end` usuario.</span><span class="sxs-lookup"><span data-stu-id="46291-111">The `Prefer: outlook.timezone` header causes the `start` and `end` of the events to be returned in the user's time zone.</span></span>
        - <span data-ttu-id="46291-112">La `Select` función limita los campos devueltos para cada evento a solo aquellos que la aplicación usará realmente.</span><span class="sxs-lookup"><span data-stu-id="46291-112">The `Select` function limits the fields returned for each event to just those the app will actually use.</span></span>
        - <span data-ttu-id="46291-113">La `OrderBy` función ordena los resultados por fecha y hora de inicio.</span><span class="sxs-lookup"><span data-stu-id="46291-113">The `OrderBy` function sorts the results by the start date and time.</span></span>
        - <span data-ttu-id="46291-114">La `Top` función solicita como máximo 50 eventos.</span><span class="sxs-lookup"><span data-stu-id="46291-114">The `Top` function requests at most 50 events.</span></span>

1. <span data-ttu-id="46291-115">Ejecuta la aplicación, inicia sesión y haz clic en el **elemento de** navegación Calendario del menú.</span><span class="sxs-lookup"><span data-stu-id="46291-115">Run the app, sign in, and click the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="46291-116">Debería ver un volcado JSON de los eventos en el calendario del usuario.</span><span class="sxs-lookup"><span data-stu-id="46291-116">You should see a JSON dump of the events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="46291-117">Mostrar los resultados</span><span class="sxs-lookup"><span data-stu-id="46291-117">Display the results</span></span>

<span data-ttu-id="46291-118">Ahora puede reemplazar el volcado json por algo para mostrar los resultados de forma fácil de usar.</span><span class="sxs-lookup"><span data-stu-id="46291-118">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span>

<span data-ttu-id="46291-119">Empiece por crear un convertidor de valores [de](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) enlace para convertir los valores [dateTimeTimeZone](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) devueltos por Microsoft Graph en los formatos de fecha y hora que espera el usuario.</span><span class="sxs-lookup"><span data-stu-id="46291-119">Start by creating a [binding value converter](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) to convert the [dateTimeTimeZone](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) values returned by Microsoft Graph into the date and time formats the user expects.</span></span>

1. <span data-ttu-id="46291-120">Haga clic con el **botón** secundario en la carpeta Modelos en el proyecto **GraphTutorial** y seleccione **Agregar**, después **Clase...**. Asigne un nombre a `GraphDateTimeTimeZoneConverter` la clase y seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="46291-120">Right-click the **Models** folder in the **GraphTutorial** project and select **Add**, then **Class...**. Name the class `GraphDateTimeTimeZoneConverter` and select **Add**.</span></span>

1. <span data-ttu-id="46291-121">Reemplace todo el contenido del archivo por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="46291-121">Replace the entire contents of the file with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GraphTutorial/Models/GraphDateTimeTimeZoneConverter.cs" id="DateTimeConverterSnippet":::

1. <span data-ttu-id="46291-122">Reemplaza todo el contenido de **CalendarPage.xaml** por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="46291-122">Replace the entire contents of **CalendarPage.xaml** with the following.</span></span>

    :::code language="xaml" source="../demo/GraphTutorial/GraphTutorial/CalendarPage.xaml" id="CalendarPageXamlSnippet":::

    <span data-ttu-id="46291-123">Esto reemplaza el `Editor` archivo con un archivo `ListView` .</span><span class="sxs-lookup"><span data-stu-id="46291-123">This replaces the `Editor` with a `ListView`.</span></span> <span data-ttu-id="46291-124">El `DataTemplate` usado para representar cada elemento usa el para convertir las propiedades y del evento en una `GraphDateTimeTimeZoneConverter` `Start` `End` cadena.</span><span class="sxs-lookup"><span data-stu-id="46291-124">The `DataTemplate` used to render each item uses the `GraphDateTimeTimeZoneConverter` to convert the `Start` and `End` properties of the event to a string.</span></span>

1. <span data-ttu-id="46291-125">Abra **CalendarPage.xaml.cs** y quite las siguientes líneas de la `OnAppearing` función.</span><span class="sxs-lookup"><span data-stu-id="46291-125">Open **CalendarPage.xaml.cs** and remove the following lines from the `OnAppearing` function.</span></span>

    ```csharp
    // Temporary
    JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
    ```

1. <span data-ttu-id="46291-126">En su lugar, agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="46291-126">In their place, add the following code.</span></span>

    ```csharp
    // Add the events to the list view
    CalendarList.ItemsSource = events.CurrentPage.ToList();
    ```

1. <span data-ttu-id="46291-127">Ejecuta la aplicación, inicia sesión y haz clic en el **elemento de** navegación Calendario.</span><span class="sxs-lookup"><span data-stu-id="46291-127">Run the app, sign in, and click the **Calendar** navigation item.</span></span> <span data-ttu-id="46291-128">Debería ver la lista de eventos con los valores **de Inicio** **y** Fin con formato.</span><span class="sxs-lookup"><span data-stu-id="46291-128">You should see the list of events with the **Start** and **End** values formatted.</span></span>

    ![Captura de pantalla de la tabla de eventos](./images/calendar-page.png)
