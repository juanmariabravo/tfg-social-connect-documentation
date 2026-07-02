# Social Connect - Documentación del Proyecto

Este repositorio reúne toda la documentación técnica, especificaciones funcionales y diagramas de diseño desarrollados para el proyecto "Social Connect", una plataforma web de conexión social basada en intereses compartidos y personalidad.

## Contenido del Repositorio

Para facilitar la navegación por la documentación del TFG, el material se organiza en los siguientes apartados:

### 1. Especificaciones Funcionales por Sprints (`SPECS.md/`)
Contiene los requisitos ágiles detallados e historias de usuario desarrollados a lo largo del ciclo de vida del proyecto, divididos en:
-   **`backend_specs/`**: Requisitos detallados de API, lógica de emparejamiento, autenticación y sockets para los Sprints 1 al 5.
-   **`frontend_specs/`**: Historias de usuario, especificaciones de vistas, interacciones de UI y casos de uso del cliente para los Sprints 1 al 5.

### 2. Diccionario de la API (`DICCIONARIO_API.md`)
Un documento técnico completo que detalla cada endpoint disponible en la API REST, detallando:
-   Métodos HTTP (`GET`, `POST`, `PUT`, `DELETE`).
-   Estructuras de datos requeridas en las peticiones (Payloads).
-   Formatos y códigos de respuesta esperados.

### 3. Diagramas y Modelado (`Figuras_y_Diagramas_TFG/`)
Directorio que recopila los modelos visuales clave del sistema en formatos editables (`.svg`, `.pdf`) y renderizados (`.png`):
-   **Arquitectura:** Diagramas detallados del backend (MVC) y frontend de la aplicación.
-   **Base de Datos:** Modelo entidad-relación y diseño lógico de las colecciones de MongoDB (`social-connect-data-modeling.png`).
-   **Casos de Uso:** Interacciones globales de los usuarios con el sistema (`CdU_SocialConnect.svg`).
-   **Flujos Críticos:** Diagrama de eventos de WebSockets para chats y notificaciones en tiempo real, así como el flujo de creación de planes.
-   **Diagrama Navegacional:** Mapeo de la navegación completa del frontend.
