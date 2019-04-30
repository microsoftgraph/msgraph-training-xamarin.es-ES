<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, creará una nueva aplicación nativa de Azure AD con el centro de administración de Azure Active Directory.

1. Abra un explorador y vaya al [centro de administración de Azure Active Directory](https://aad.portal.azure.com) e inicie sesión con una **cuenta personal** (también conocida como: cuenta Microsoft) o una **cuenta profesional o educativa**.

1. Seleccione **Azure Active Directory** en el panel de navegación izquierdo y seleccione **Registros de aplicaciones (versión preliminar)** en **Administrar**.

    ![Una captura de pantalla de los registros de la aplicación ](./images/aad-portal-app-registrations.png)

1. Seleccione **Nuevo registro**. En la página **Registrar una aplicación**, establezca los valores siguientes.

    - Establezca **Nombre** como `Xamarin Graph Tutorial`.
    - Establezca **Tipos de cuenta admitidos** en **Cuentas en cualquier directorio de organización y cuentas personales de Microsoft**.
    - Deje **URI de redireccionamiento** vacía.

    ![Captura de pantalla de la página registrar una aplicación](./images/aad-register-an-app.png)

1. Elija **Registrar**. En la página **tutorial de Xamarin Graph** , copie el valor del **identificador de la aplicación (cliente)** y guárdelo, lo necesitará en el paso siguiente.

    ![Captura de pantalla del identificador de la aplicación del nuevo registro de la aplicación](./images/aad-application-id.png)

1. Seleccione el vínculo **Agregar un URI de** redireccionamiento. En la página **URI** de redireccionamiento, busque la sección **URI de redireccionamiento sugeridos para clientes públicos (móvil, escritorio)** . Seleccione el URI que comienza con `msal` **y** el **urn: IETF: WG: OAuth: 2.0:** el URI de OOB. Copie el valor que comienza con `msal`y, a continuación, elija **Guardar**. Guarde el URI de redireccionamiento copiado y lo necesitará en el paso siguiente.

    ![Captura de pantalla de la página URI de redireccionamiento](./images/aad-redirect-uris.png)