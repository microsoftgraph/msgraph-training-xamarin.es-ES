<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="acd79-101">En este ejercicio, incorporará Microsoft Graph a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="acd79-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="acd79-102">Para esta aplicación, usará la [biblioteca de cliente de Microsoft Graph para .net](https://github.com/microsoftgraph/msgraph-sdk-dotnet) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="acd79-102">For this application, you will use the [Microsoft Graph Client Library for .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="acd79-103">Obtener eventos de calendario de Outlook</span><span class="sxs-lookup"><span data-stu-id="acd79-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="acd79-104">Para empezar, actualice el archivo **CalendarPage. Xaml** en el proyecto **GraphTutorial** .</span><span class="sxs-lookup"><span data-stu-id="acd79-104">Start by updating the **CalendarPage.xaml** file in the **GraphTutorial** project.</span></span> <span data-ttu-id="acd79-105">Abra el archivo y reemplace el contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="acd79-105">Open the file and replace its contents with the following.</span></span>

```xml
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

<span data-ttu-id="acd79-106">Abra `CalendarPage.xaml.cs` y agregue las siguientes `using` instrucciones en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="acd79-106">Open `CalendarPage.xaml.cs` and add the following `using` statements at the top of the file.</span></span>

```cs
using Newtonsoft.Json;
using Microsoft.Graph;
using System.ComponentModel;
using System.Collections.ObjectModel;
```

<span data-ttu-id="acd79-107">A continuación, agregue el siguiente código para obtener los eventos del usuario y mostrar la respuesta JSON.</span><span class="sxs-lookup"><span data-stu-id="acd79-107">Then add the following code to get the user's events and display the JSON response.</span></span>

```cs
protected override async void OnAppearing()
{
    base.OnAppearing();

    // Get the events
    var events = await App.GraphClient.Me.Events.Request()
        .Select("subject,organizer,start,end")
        .OrderBy("createdDateTime DESC")
        .GetAsync();

    // Temporary
    JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
}
```

<span data-ttu-id="acd79-108">Tenga en cuenta lo que `OnAppearing` hace el código.</span><span class="sxs-lookup"><span data-stu-id="acd79-108">Consider what the code in `OnAppearing` is doing.</span></span>

- <span data-ttu-id="acd79-109">La dirección URL a la que se `/v1.0/me/events`llamará es.</span><span class="sxs-lookup"><span data-stu-id="acd79-109">The URL that will be called is `/v1.0/me/events`.</span></span>
- <span data-ttu-id="acd79-110">La `Select` función limita los campos devueltos para cada evento a solo aquellos que la vista usará realmente.</span><span class="sxs-lookup"><span data-stu-id="acd79-110">The `Select` function limits the fields returned for each events to just those the view will actually use.</span></span>
- <span data-ttu-id="acd79-111">La `OrderBy` función ordena los resultados por la fecha y hora en que se crearon, con el elemento más reciente en primer lugar.</span><span class="sxs-lookup"><span data-stu-id="acd79-111">The `OrderBy` function sorts the results by the date and time they were created, with the most recent item being first.</span></span>

<span data-ttu-id="acd79-112">Ahora puede ejecutar la aplicación, iniciar sesión y hacer clic en el elemento de navegación **calendario** en el menú.</span><span class="sxs-lookup"><span data-stu-id="acd79-112">You can now run the app, sign in, and click the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="acd79-113">Debería ver un volcado JSON de los eventos en el calendario del usuario.</span><span class="sxs-lookup"><span data-stu-id="acd79-113">You should see a JSON dump of the events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="acd79-114">Mostrar los resultados</span><span class="sxs-lookup"><span data-stu-id="acd79-114">Display the results</span></span>

<span data-ttu-id="acd79-115">Ahora puede reemplazar el volcado de JSON con algo para mostrar los resultados de forma fácil de uso.</span><span class="sxs-lookup"><span data-stu-id="acd79-115">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="acd79-116">Para empezar, cree un [convertidor de valores de enlace](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) para convertir los valores de [dateTimeTimeZone](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) devueltos por Microsoft Graph en los formatos de fecha y hora que el usuario espera.</span><span class="sxs-lookup"><span data-stu-id="acd79-116">Start by creating a [binding value converter](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) to convert the [dateTimeTimeZone](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) values returned by Microsoft Graph into the date and time formats the user expects.</span></span> <span data-ttu-id="acd79-117">Haga clic con el botón derecho en la carpeta **modelos** en el proyecto **GraphTutorial** y elija **Agregar**y, a continuación, **clase...**. Asigne un nombre `GraphDateTimeTimeZoneConverter` a la clase y elija **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="acd79-117">Right-click the **Models** folder in the **GraphTutorial** project and choose **Add**, then **Class...**. Name the class `GraphDateTimeTimeZoneConverter` and choose **Add**.</span></span> <span data-ttu-id="acd79-118">Reemplace todo el contenido del archivo por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="acd79-118">Replace the entire contents of the file with the following.</span></span>

```cs
using Microsoft.Graph;
using System;
using System.Globalization;
using Xamarin.Forms;

namespace GraphTutorial.Models
{
    class GraphDateTimeTimeZoneConverter : IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
        {
            if (value is DateTimeTimeZone date)
            {
                // Resolve the time zone
                var timezone = TimeZoneInfo.FindSystemTimeZoneById(date.TimeZone);
                // Parse method assumes local time, which may not be the case
                var parsedDateAsLocal = DateTimeOffset.Parse(date.DateTime);
                // Determine the offset from UTC time for the specific date
                // Making this call adjusts for DST as appropriate
                var tzOffset = timezone.GetUtcOffset(parsedDateAsLocal.DateTime);
                // Create a new DateTimeOffset with the specific offset from UTC
                var correctedDate = new DateTimeOffset(parsedDateAsLocal.DateTime, tzOffset);
                // Return the local date time string
                return $"{correctedDate.LocalDateTime.ToShortDateString()} {correctedDate.LocalDateTime.ToShortTimeString()}";
            }

            return string.Empty;
        }

        public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        {
            throw new NotImplementedException();
        }
    }
}
```

<span data-ttu-id="acd79-119">A continuación, reemplace todo el contenido `CalendarPage.xaml` de con lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="acd79-119">Next, replace the entire contents of `CalendarPage.xaml` with the following.</span></span>

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:local="clr-namespace:GraphTutorial.Models"
             Title="Calendar"
             x:Class="GraphTutorial.CalendarPage">
    <ContentPage.Resources>
        <local:GraphDateTimeTimeZoneConverter x:Key="DateConverter" />
    </ContentPage.Resources>
    <ContentPage.Content>
        <StackLayout>
            <ListView x:Name="CalendarList"
                      HasUnevenRows="true"
                      Margin="10,10,10,10">
                <ListView.ItemTemplate>
                    <DataTemplate>
                        <ViewCell>
                            <StackLayout Margin="10,10,10,10">
                                <Label Text="{Binding Path=Subject}"
                                       FontAttributes="Bold"
                                       FontSize="Medium" />
                                <Label Text="{Binding Path=Organizer.EmailAddress.Name}"
                                       FontSize="Small" />
                                <StackLayout Orientation="Horizontal">
                                    <Label Text="{Binding Path=Start, Converter={StaticResource DateConverter}}"
                                       FontSize="Micro" />
                                    <Label Text="to"
                                           FontSize="Micro" />
                                    <Label Text="{Binding Path=End, Converter={StaticResource DateConverter}}"
                                       FontSize="Micro" />
                                </StackLayout>
                            </StackLayout>
                        </ViewCell>
                    </DataTemplate>
                </ListView.ItemTemplate>
            </ListView>
        </StackLayout>
    </ContentPage.Content>
</ContentPage>
```

<span data-ttu-id="acd79-120">Esto reemplaza `Editor` por un `ListView`.</span><span class="sxs-lookup"><span data-stu-id="acd79-120">This replaces the `Editor` with a `ListView`.</span></span> <span data-ttu-id="acd79-121">El `DataTemplate` que se usa para representar cada elemento `GraphDateTimeTimeZoneConverter` utiliza el para `Start` convertir `End` las propiedades y del evento en una cadena.</span><span class="sxs-lookup"><span data-stu-id="acd79-121">The `DataTemplate` used to render each item uses the `GraphDateTimeTimeZoneConverter` to convert the `Start` and `End` properties of the event to a string.</span></span> <span data-ttu-id="acd79-122">Ahora, `CalendarPage.xaml.cs` abra y quite las siguientes líneas de `OnAppearing` la función.</span><span class="sxs-lookup"><span data-stu-id="acd79-122">Now open `CalendarPage.xaml.cs` and remove the following lines from the `OnAppearing` function.</span></span>

```cs
// Temporary
JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
```

<span data-ttu-id="acd79-123">En su ubicación, agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="acd79-123">In their place, add the following code.</span></span>

```cs
// Add the events to the list view
CalendarList.ItemsSource = events.CurrentPage.ToList();
```

<span data-ttu-id="acd79-124">Ejecute la aplicación, inicie sesión y haga clic en el elemento de navegación **calendario** .</span><span class="sxs-lookup"><span data-stu-id="acd79-124">Run the app, sign in, and click the **Calendar** navigation item.</span></span> <span data-ttu-id="acd79-125">Debe ver la lista de eventos con los valores de **Inicio** y **finalización** con formato.</span><span class="sxs-lookup"><span data-stu-id="acd79-125">You should see the list of events with the **Start** and **End** values formatted.</span></span>

![Captura de pantalla de la tabla de eventos](./images/calendar-page.png)
