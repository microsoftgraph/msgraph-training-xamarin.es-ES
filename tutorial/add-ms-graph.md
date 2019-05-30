<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, incorporará Microsoft Graph a la aplicación. Para esta aplicación, usará la [biblioteca de cliente de Microsoft Graph para .net](https://github.com/microsoftgraph/msgraph-sdk-dotnet) para realizar llamadas a Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Obtener eventos de calendario de Outlook

Para empezar, actualice el archivo **CalendarPage. Xaml** en el proyecto **GraphTutorial** . Abra el archivo y reemplace el contenido por lo siguiente.

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

Abra `CalendarPage.xaml.cs` y agregue las siguientes `using` instrucciones en la parte superior del archivo.

```cs
using Newtonsoft.Json;
using Microsoft.Graph;
using System.ComponentModel;
using System.Collections.ObjectModel;
```

A continuación, agregue el siguiente código para obtener los eventos del usuario y mostrar la respuesta JSON.

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

Tenga en cuenta lo que `OnAppearing` hace el código.

- La dirección URL a la que se `/v1.0/me/events`llamará es.
- La `Select` función limita los campos devueltos para cada evento a solo aquellos que la vista usará realmente.
- La `OrderBy` función ordena los resultados por la fecha y hora en que se crearon, con el elemento más reciente en primer lugar.

Ahora puede ejecutar la aplicación, iniciar sesión y hacer clic en el elemento de navegación **calendario** en el menú. Debería ver un volcado JSON de los eventos en el calendario del usuario.

## <a name="display-the-results"></a>Mostrar los resultados

Ahora puede reemplazar el volcado de JSON con algo para mostrar los resultados de forma fácil de uso. Para empezar, cree un [convertidor de valores de enlace](/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics#binding-value-converters) para convertir los valores de [dateTimeTimeZone](/graph/api/resources/datetimetimezone?view=graph-rest-1.0) devueltos por Microsoft Graph en los formatos de fecha y hora que el usuario espera. Haga clic con el botón derecho en la carpeta **modelos** en el proyecto **GraphTutorial** y elija **Agregar**y, a continuación, **clase...**. Asigne un nombre `GraphDateTimeTimeZoneConverter` a la clase y elija **Agregar**. Reemplace todo el contenido del archivo por lo siguiente.

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

A continuación, reemplace todo el contenido `CalendarPage.xaml` de con lo siguiente.

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

Esto reemplaza `Editor` por un `ListView`. El `DataTemplate` que se usa para representar cada elemento `GraphDateTimeTimeZoneConverter` utiliza el para `Start` convertir `End` las propiedades y del evento en una cadena. Ahora, `CalendarPage.xaml.cs` abra y quite las siguientes líneas de `OnAppearing` la función.

```cs
// Temporary
JSONResponse.Text = JsonConvert.SerializeObject(events.CurrentPage, Formatting.Indented);
```

En su ubicación, agregue el siguiente código.

```cs
// Add the events to the list view
CalendarList.ItemsSource = events.CurrentPage.ToList();
```

Ejecute la aplicación, inicie sesión y haga clic en el elemento de navegación **calendario** . Debe ver la lista de eventos con los valores de **Inicio** y **finalización** con formato.

![Captura de pantalla de la tabla de eventos](./images/calendar-page.png)
