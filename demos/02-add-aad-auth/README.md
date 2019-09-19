# <a name="completed-module-add-azure-ad-authentication"></a>Módulo completado: agregar autenticación de Azure AD

La versión del proyecto en este directorio refleja cómo completar el tutorial hasta [Agregar autenticación de Azure ad](https://docs.microsoft.com/graph/tutorials/xamarin?tutorial-step=3). Si usa esta versión del proyecto, deberá completar el resto del tutorial a partir de [obtener datos de calendario](https://docs.microsoft.com/graph/tutorials/xamarin?tutorial-step=4).

> **Nota:** Se supone que ya ha registrado una aplicación en Azure portal como se especifica en [registrar la aplicación en el portal](https://docs.microsoft.com/graph/tutorials/xamarin?tutorial-step=2). Debe configurar esta versión del ejemplo de la siguiente manera:
>
> 1. Cambie el nombre `./GraphTutorial/GraphTutorial/Models/OAuthSettings.cs.example` del archivo `OAuthSettings.cs`a.
> 1. Edite `OAuthSettings.cs` el archivo y realice los cambios siguientes.
>     1. Reemplace `YOUR_APP_ID_HERE` por el **identificador de aplicación** que obtuvo desde el portal de Azure.
> 1. En el proyecto **GraphTutorial. iOS** , abra el `Info.plist` archivo.
>    1. En la ficha **avanzadas** , busque la sección **tipos de URL** . En el valor **esquemas** de URL `YOUR_APP_ID_HERE` , reemplace por el **identificador de aplicación** que obtuvo desde el portal de Azure.
> 1. En el proyecto **GraphTutorial. Android** , abra el `Properties/AndroidManifest.xml` archivo.
>     1. Reemplace `YOUR_APP_ID_HERE` por el **identificador de aplicación** que obtuvo desde el portal de Azure.
