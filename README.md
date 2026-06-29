# Keeperly

Keeperly es una aplicación móvil nativa para Android diseñada para simplificar y automatizar la gestión de finanzas personales. Resuelve el problema del registro manual tedioso de gastos mediante la sincronización automatizada con cuentas de PayPal Sandbox, permitiendo a los usuarios planificar presupuestos personalizados, organizar transacciones y realizar un seguimiento claro de sus metas financieras en tiempo real.

# Descripción detallada

Keeperly nace de un proyecto la Facultad de Informática de la Universidad Complutense de Madrid (UCM) como una solución de la necesidad de controlar los gastos diarios de forma rápida, segura y organizada. A menudo, las personas abandonan el control de sus finanzas personales debido al esfuerzo manual que requiere anotar cada café, compra o transferencia. 

Esta aplicación ofrece un entorno centralizado donde los usuarios pueden estructurar sus finanzas mediante presupuestos vinculados a categorías personalizadas (por ejemplo, alimentación, transporte, ocio). A través de una interfaz intuitiva, Keeperly muestra de forma visual la cantidad gastada, el saldo restante y los límites temporales de cada presupuesto.

Además, Keeperly cuenta con una integración con la API de PayPal Sandbox (entorno de pruebas para desarrolladores y comercios). Esto permite vincular cuentas de PayPal Business para obtener y actualizar de forma automatizada los balances de las cuentas correspondientes. Cuenta con un modo offline que almacena localmente todos los datos (a excepción de las operaciones que requieren sincronización de red con PayPal), asegurando una disponibilidad de la información y la capacidad de registrar transacciones en cualquier momento.

## Índice
- [Requisitos Previos](#requisitos-previos)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Ejecución y Uso](#ejecución-y-uso)
- [Arquitectura y Stack Tecnológico](#arquitectura-y-stack-tecnológico)
- [Contribuciones](#contribuciones)
- [Licencia y Créditos](#licencia-y-créditos)

## Requisitos Previos
Antes de comenzar con la instalación, asegúrate de cumplir con los siguientes requisitos de entorno y software:
- **Sistema Operativo recomendado:** Linux (Ubuntu 22.04+), macOS (13+ Ventura) o Windows 10/11.
- **Runtime o Lenguaje:** Java Development Kit (JDK) v17 o superior (requerido para compatibilidad con Gradle v8.5+ y Android compileSdk 34).
- **Gestor de paquetes / compilador:** Gradle Wrapper (incluido en el proyecto).
- **Dependencias del sistema:**
  - Android Studio (versión Jellyfish 2023.3.1 o superior recomendada) o Android SDK Command-line tools.
  - Dispositivo físico Android (con Depuración por USB habilitada) o un Emulador Android (AVD) configurado con un nivel de API mínimo 24 (Android 7.0 Nougat).
  - Conexión a internet (para sincronización inicial de dependencias de Gradle y la integración opcional con PayPal).

## Instalación
Sigue estos pasos para clonar el repositorio e instalar las dependencias locales:
1. Clonar el repositorio:
```bash
git clone https://github.com/Melendo/Keeperly.git
cd Keeperly
```

2. Instalar y sincronizar dependencias del proyecto:
Para descargar las dependencias Gradle del proyecto y compilarlo sin abrir Android Studio, ejecuta el siguiente comando en la raíz del proyecto:
```bash
./gradlew build
```
*(Nota: Si utilizas Windows, ejecuta `gradlew.bat build`)*. Si prefieres trabajar de forma visual, puedes abrir la carpeta del proyecto directamente en Android Studio, y la sincronización de dependencias comenzará de forma automática.

## Configuración
A diferencia de proyectos backend tradicionales, esta aplicación nativa Android no utiliza un archivo de variables de entorno `.env` en el almacenamiento local del dispositivo móvil por motivos de seguridad y arquitectura del cliente. En su lugar, la configuración e integración con los servicios externos se gestiona dinámicamente desde la interfaz de la aplicación:

### Configuración de la Base de Datos Local
La base de datos SQLite se crea e inicializa localmente de forma automática bajo el nombre `keeperly_database` en el momento del primer inicio de la aplicación (`LoginActivity`). No se requiere configuración manual.

### Credenciales de PayPal Sandbox
Para realizar la sincronización con PayPal:
1. Inicia sesión en la aplicación o regístrate para crear un nuevo perfil de usuario local.
2. Navega a la sección **Cuentas** y selecciona la cuenta que deseas sincronizar.
3. Haz clic en el botón **Sincronizar con PayPal**.
4. En el diálogo emergente, introduce las credenciales obtenidas de tu panel de desarrollador de PayPal:
   - **Client ID:** Identificador único de tu aplicación sandbox.
   - **Secret:** Clave secreta asociada al identificador.

Estas credenciales se envían de forma segura a través de HTTPS mediante OkHttp a los endpoints de PayPal Sandbox (`https://api-m.sandbox.paypal.com`) para realizar la autenticación OAuth 2.0 y obtener un token de acceso dinámico (`access_token`) con el cual consultar el balance en tiempo real de forma segura.

## Ejecución y Uso
En esta sección se explica cómo poner en marcha el proyecto en las distintas fases del ciclo de desarrollo.

### Entorno de Desarrollo
Para compilar e instalar la aplicación en modo de depuración (debug) en un emulador activo o dispositivo Android conectado físicamente, ejecuta:
```bash
./gradlew installDebug
```
Una vez finalizada la instalación, la aplicación se iniciará de forma automática en el dispositivo/emulador. Alternativamente, puedes hacer clic en el botón **Run** (icono verde de reproducción) dentro de Android Studio.

### Pruebas (Testing)
Para ejecutar la suite de pruebas automatizadas en el proyecto:

1. **Pruebas Unitarias Locales (JUnit):**
Ejecuta la suite de pruebas de lógica de negocio y repositorios en tu entorno local sin requerir un emulador Android:
```bash
./gradlew test
```

2. **Pruebas de Instrumentación / Integración (Espresso):**
Asegúrate de tener un emulador en ejecución o un dispositivo conectado y ejecuta:
```bash
./gradlew connectedAndroidTest
```

## Arquitectura y Stack Tecnológico
Keeperly está estructurado bajo el patrón de diseño **MVVM (Model-View-ViewModel)** recomendado por Google para Android, garantizando la separación de responsabilidades y facilitando la mantenibilidad:

- **Frontend / UI (View):** Activities (`LoginActivity`, `RegisterActivity`, `MenuActivity`) y Fragments (`InicioFragment`, `CuentasFragment`, `PresupuestosFragment`, etc.) desarrollados mediante XML layouts nativos con soporte para **ViewBinding** para una vinculación segura y limpia de las vistas.
- **Intermediarios (ViewModel):** Clases ViewModel correspondientes para gestionar el estado de la UI y los flujos de datos asíncronos mediante **LiveData** (comunicando cambios reactivos de la base de datos a las vistas).
- **Capa de Datos (Model):** 
  - **Persistencia Local:** SQLite integrado mediante la biblioteca de persistencia **Android Room** (con DAOs y entidades mapeadas en POJOs de Java con convertidor de tipos `DateConverter` para el manejo de fechas).
  - **Acceso a Datos:** Repositorios (`UsuarioRepository`, `CuentaRepository`, `CategoriaRepository`, `PresupuestoRepository`, `TransaccionRepository`) centralizados a través de un singleton de control (`RepositoryFactory`).
- **Conectividad / API:** **OkHttp** para peticiones de red asíncronas con gestión de callbacks, interactuando con la API REST de PayPal Sandbox para la autenticación y obtención de saldos.
- **Modo Offline:** Monitoreo dinámico del estado de conectividad mediante `ConnectivityManager` y `NetworkCallback`, bloqueando de forma segura las funciones de red (como PayPal Sync) mientras se mantiene la operatividad local de transacciones y presupuestos.

## Contribuciones
Si deseas contribuir al desarrollo de este proyecto, por favor sigue los siguientes pasos:
1. Haz un Fork del repositorio.
2. Crea una nueva rama para tu funcionalidad (`git checkout -b feature/nueva-funcionalidad`).
3. Realiza tus cambios y haz un commit claro siguiendo convenciones (`git commit -m 'Añade nueva funcionalidad'`).
4. Sube los cambios a tu rama (`git push origin feature/nueva-funcionalidad`).
5. Abre una Pull Request explicando detalladamente los cambios realizados.

## Licencia y Créditos

### Créditos y Agradecimientos
- Desarrollado por [Melendo](https://github.com/Melendo), [BeaBueno](https://github.com/beabuenodev), [Iván Alcalde](https://github.com/IAlcCamDev), [Amaia](https://github.com/amaiaech) y [Gonzalo](https://github.com/gonzaloasecas).
- Creado en el marco académico de la Facultad de Informática, Universidad Complutense de Madrid (UCM).

---

## Información de la plantilla
> [!TIP]
> Al adaptar esta plantilla a tu proyecto, añade o elimina los apartados que consideres necesarios. El objetivo es mantener la documentación lo más simple, clara y concisa posible, omitiendo las secciones que no apliquen.
