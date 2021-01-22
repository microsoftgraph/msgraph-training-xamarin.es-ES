<!-- markdownlint-disable MD002 MD041 -->

En este tutorial se explica cómo crear una aplicación xamarin que use la API de Microsoft Graph para recuperar la información de calendario de un usuario.

> [!TIP]
> Si prefiere descargar el tutorial completado, puede descargar o clonar el [repositorio de GitHub.](https://github.com/microsoftgraph/msgraph-training-xamarin)

## <a name="prerequisites"></a>Requisitos previos

Antes de iniciar este tutorial, debes tener [Visual Studio](https://visualstudio.microsoft.com/vs/) en un equipo que ejecute Windows 10 con el modo [de desarrollador activado.](https://docs.microsoft.com/windows/uwp/get-started/enable-your-device-for-development) Si no tiene una Visual Studio, visite el vínculo anterior para ver las opciones de descarga.

También debe tener Xamarin instalado como parte de la Visual Studio instalación. Consulte [Instalar Xamarin para](/xamarin/cross-platform/get-started/installation) obtener instrucciones sobre cómo instalar y configurar Xamarin.

Opcionalmente, también debes tener acceso a un equipo Mac con Visual Studio para Mac instalado. Si no tiene acceso, todavía puede completar este tutorial, pero no podrá completar las secciones específicas de iOS.

También debe tener una cuenta personal de Microsoft con un buzón en Outlook.com o una cuenta de Microsoft para el trabajo o la escuela. Si no tienes una cuenta de Microsoft, hay un par de opciones para obtener una cuenta gratuita:

- Puede registrarse [para obtener una nueva cuenta personal de Microsoft.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)
- Puede registrarse en el Programa de desarrolladores de [Office 365](https://developer.microsoft.com/office/dev-program) para obtener una suscripción gratuita a Office 365.

> [!NOTE]
> Este tutorial se escribió con Visual Studio 2019 versión 16.8.3 y Visual Studio para Mac versión 8.5.1. Ambas máquinas tienen instalada la plataforma 28 del SDK de Android. Los pasos de esta guía pueden funcionar con otras versiones, pero no se han probado.

## <a name="feedback"></a>Comentarios

Envíe sus comentarios sobre este tutorial en el [repositorio de GitHub.](https://github.com/microsoftgraph/msgraph-training-xamarin)
