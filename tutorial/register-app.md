<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="69d87-101">En este ejercicio, creará una nueva aplicación nativa de Azure AD con el centro de administración de Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="69d87-101">In this exercise you will create a new Azure AD native application using the Azure Active Directory admin center.</span></span>

1. <span data-ttu-id="69d87-102">Abra un explorador y vaya al [centro de administración de Azure Active Directory](https://aad.portal.azure.com) e inicie sesión con una **cuenta personal** (también conocida como: cuenta Microsoft) o una **cuenta profesional o educativa**.</span><span class="sxs-lookup"><span data-stu-id="69d87-102">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com) and login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="69d87-103">Seleccione **Azure Active Directory** en el panel de navegación de la izquierda y, después, seleccione **registros de aplicaciones** en **administrar**.</span><span class="sxs-lookup"><span data-stu-id="69d87-103">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.</span></span>

    ![<span data-ttu-id="69d87-104">Una captura de pantalla de los registros de la aplicación</span><span class="sxs-lookup"><span data-stu-id="69d87-104">A screenshot of the App registrations</span></span> ](./images/aad-portal-app-registrations.png)

1. <span data-ttu-id="69d87-105">Seleccione **Nuevo registro**.</span><span class="sxs-lookup"><span data-stu-id="69d87-105">Select **New registration**.</span></span> <span data-ttu-id="69d87-106">En la página **Registrar una aplicación**, establezca los valores siguientes.</span><span class="sxs-lookup"><span data-stu-id="69d87-106">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="69d87-107">Establezca **Nombre** como `Xamarin Graph Tutorial`.</span><span class="sxs-lookup"><span data-stu-id="69d87-107">Set **Name** to `Xamarin Graph Tutorial`.</span></span>
    - <span data-ttu-id="69d87-108">Establezca **Tipos de cuenta admitidos** en **Cuentas en cualquier directorio de organización y cuentas personales de Microsoft**.</span><span class="sxs-lookup"><span data-stu-id="69d87-108">Set **Supported account types** to **Accounts in any organizational directory and personal Microsoft accounts**.</span></span>
    - <span data-ttu-id="69d87-109">En **URI de redireccionamiento (opcional)**, cambie la lista desplegable a **cliente público (móvil & escritorio)** y establezca `urn:ietf:wg:oauth:2.0:oob`el valor en.</span><span class="sxs-lookup"><span data-stu-id="69d87-109">Under **Redirect URI (optional)**, change the dropdown to **Public client (mobile & desktop)**, and set the value to `urn:ietf:wg:oauth:2.0:oob`.</span></span>

    ![Captura de pantalla de la página registrar una aplicación](./images/aad-register-an-app.png)

1. <span data-ttu-id="69d87-111">Elija **Registrar**.</span><span class="sxs-lookup"><span data-stu-id="69d87-111">Choose **Register**.</span></span> <span data-ttu-id="69d87-112">En la página **tutorial de Xamarin Graph** , copie el valor del **identificador de la aplicación (cliente)** y guárdelo, lo necesitará en el paso siguiente.</span><span class="sxs-lookup"><span data-stu-id="69d87-112">On the **Xamarin Graph Tutorial** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

    ![Captura de pantalla del identificador de la aplicación del nuevo registro de la aplicación](./images/aad-application-id.png)

1. <span data-ttu-id="69d87-114">Seleccione la **autenticación** en **administrar**.</span><span class="sxs-lookup"><span data-stu-id="69d87-114">Select the **Authentication** under **Manage**.</span></span> <span data-ttu-id="69d87-115">Busque la sección **de los URI de redireccionamiento sugeridos para clientes públicos (móvil, escritorio)** .</span><span class="sxs-lookup"><span data-stu-id="69d87-115">Locate the **Suggested Redirect URIs for public clients (mobile, desktop)** section.</span></span> <span data-ttu-id="69d87-116">Seleccione el URI que comienza con `msal`y, a continuación, elija **Guardar**.</span><span class="sxs-lookup"><span data-stu-id="69d87-116">Select the URI that begins with `msal`, then choose **Save**.</span></span>

    ![Captura de pantalla de la página URI de redireccionamiento](./images/aad-redirect-uris.png)
