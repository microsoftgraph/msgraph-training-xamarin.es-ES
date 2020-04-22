<!-- markdownlint-disable MD002 MD041 -->

Este tutorial le enseña a crear una aplicación Xamarin que use la API de Microsoft Graph para recuperar la información de calendario de un usuario.

> [!TIP]
> Si prefiere descargar solo el tutorial completo, puede descargar o clonar el repositorio de [GitHub](https://github.com/microsoftgraph/msgraph-training-xamarin).

## <a name="prerequisites"></a>Requisitos previos

Antes de iniciar este tutorial, debe tener [Visual Studio](https://visualstudio.microsoft.com/vs/) instalado en un equipo que ejecute Windows 10 con el [modo de programador activado](https://docs.microsoft.com/windows/uwp/get-started/enable-your-device-for-development). Si no tiene Visual Studio, visite el vínculo anterior de opciones de descarga.

También debe tener Xamarin instalado como parte de la instalación de Visual Studio. Consulte [instalar Xamarin](/xamarin/cross-platform/get-started/installation) para obtener instrucciones sobre cómo instalar y configurar Xamarin.

Opcionalmente, también debe tener acceso a un equipo Mac con Visual Studio para Mac instalado. Si no tiene acceso, puede completar este tutorial, pero no podrá completar las secciones específicas de iOS.

También debe tener una cuenta de Microsoft personal con un buzón de correo en Outlook.com o una cuenta profesional o educativa de Microsoft. Si no tiene una cuenta de Microsoft, hay un par de opciones para obtener una cuenta gratuita:

- Puede [registrarse para obtener una nueva cuenta Microsoft personal](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).
- Puede [registrarse para el programa de desarrolladores de office 365](https://developer.microsoft.com/office/dev-program) para obtener una suscripción gratuita a Office 365.

> [!NOTE]
> Este tutorial se ha escrito con Visual Studio 2019 versión 16.5.2 y Visual Studio para Mac versión 8.5.1. Ambos equipos tienen instalada la plataforma SDK de Android 28. Los pasos de esta guía pueden funcionar con otras versiones, pero no se han probado.

## <a name="feedback"></a>Comentarios

Envíe sus comentarios sobre este tutorial en el [repositorio de github](https://github.com/microsoftgraph/msgraph-training-xamarin).
