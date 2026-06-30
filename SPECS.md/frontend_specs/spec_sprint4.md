# Especificación Técnica: Sprint 4 - Frontend: Planes, Descubrimiento y Notificaciones
**Fechas:** 01/06/2026 - 14/06/2026
**Estado:** Finalizado ✅

---

## 0. Fase de Diseño (Mockups)
**Herramienta:** Figma / Lovable
**Objetivo:** Crear las vistas de alta fidelidad para conectarlas con la lógica del backend.

### 0.1 Mockups a Implementar
- [x] **Home (Descubrimiento):** Swipe cards para conectar con nuevos usuarios.
- [x] **Planes (Feed):** Listado interactivo con avatares de participantes y barra de reacciones.
- [x] **Planes (Creación):** Formulario para lanzar propuestas rápidas.
- [x] **Notificaciones:** Vista centralizada con indicadores de actividad reciente.
- [x] **Perfil (Gestión):** Sección para listar/eliminar amigos y botón de Logout.

---

## 1. Alcance del Sprint
El objetivo es diseñar el prototipo estático y transformarlo en una aplicación funcional, integrando el sistema de planes, las notificaciones en tiempo real vía Sockets, el flujo de descubrimiento de usuarios y la gestión de sesión/amistades.

## 2. Requisitos Técnicos
- **Framework:** React 19 (Vite / TS)
- **Estilos:** Tailwind CSS 4 + Framer Motion (para animaciones de swipe).
- **WebSockets:** `socket.io-client` para notificaciones push y actividad en tiempo real.
- **Navegación:** React Router.

## 3. Vistas y Componentes Clave

### 3.1 Inicio / Descubrimiento (`src/pages/Home.tsx`)
- **Funcionalidad:**
  - Carga de perfiles aleatorios (`GET /api/profiles/random`).
  - Lógica de **Swipe Cards** con `framer-motion` (Derecha: Conectar, Izquierda: Descartar).
  - Acción "Conectar" (❤️): Llama a `POST /api/friends/request/:id`.
  - Acción "Descartar" (❌): Animación de salida y carga del siguiente perfil.
  - Soporte para fotos "Real" y "Virtual" con selector de tipo dinámico por tarjeta.

### 3.2 Planes - Feed y Creación (`src/pages/Plans.tsx`)
- **Funcionalidad:**
  - Carga de planes con `GET /api/plans`.
  - **Reacciones:** Actualización instantánea del contador al pulsar botón de reacciones y llamada a `POST /api/plans/:id/react`.
  - **Participación:** Al pulsar "Me apunto", añadir avatar dinámicamente y hacer la llamada a `POST /api/plans/:id/join`.
  - **Comentarios:** Hilo de discusión por plan. Al enviar un comentario, se actualiza el feed y se notifica al creador del plan. Se hace llamada a `POST /api/plans/:id/comments`.
  - **Formulario:** Modal de creación con validación de campos para la creación de planes con `POST /api/plans`.

### 3.3 Notificaciones (`src/pages/Notifications.tsx`)
- **Descripción**: Centro de avisos e indicador numérico en la navegación.
- **Funcionalidad:**
  - En `AppShell.tsx`: Consultar cantidad de notificaciones no leídas (`GET /api/notifications/unread-count`).
  - En `Notifications`: Listar datos reales de la API. Marcar como leídas visualmente si `read === true`.
  - **Gestión Directa:** Aceptar o rechazar solicitudes de amistad directamente desde la notificación.
  - Redirección lógica al hacer click según el `type` de notificación.

### 3.4 Perfil y Gestión de Amigos (`src/pages/Friends.tsx` y `src/pages/Profile.tsx`)
- **Descripción**: Visualización del listado de amigos y cierre de sesión.
- **Funcionalidad:**
  - **Amigos (`Friends.tsx`):** Listado detallado de amistades con búsqueda integrada accesible desde el perfil. Cada amigo tiene un botón de eliminar que llama a `DELETE /api/friends/:id` con confirmación.
  - **Eliminar Amigo:** Botón con diálogo de confirmación que llama a `DELETE /api/friends/:id`.
  - **Cierre de Sesión:** Botón de Logout en `AppShell` que limpia el contexto de autenticación y redirige a `/`.

## 4. Integración Síncrona (WebSockets)
- **Tecnología:** Context API `SocketContext.tsx`.
- **Eventos a Escuchar:**
  - `new_notification`: Actualiza el contador de notificaciones en tiempo real y refresca la lista si el usuario está en la vista.
  - `new_message`: Actualiza el indicador de chats no leídos.
  - `plan_join` / `plan_comment`: Notifica si hay actividad en planes.

## 5. Definición de Hecho (Definition of Done)
- [x] Vistas creadas y conectadas con los endpoints del backend.
- [x] Lógica de swipe en el Home operativa con llamadas al backend.
- [x] Feed de planes con soporte para reacciones y comentarios funcionales.
- [x] Centro de notificaciones reactivo a eventos de Socket.io.
- [x] Gestión de amigos (listar y eliminar) implementada.
- [x] Botón de cierre de sesión funcional.
- [x] UI adaptada (Responsive) para móvil y escritorio.
- [x] Código verificado y estable.

*La documentación refleja la realidad del código al cierre del Sprint 4.*
