# Especificación Técnica: Sprint 4 - Backend: Sistema de Planes y Notificaciones
**Fechas:** 01/06/2026 - 14/06/2026
**Estado:** Finalizado ✅

---

## 1. Alcance del Sprint
El objetivo es implementar la "excusa para hablar" mediante un sistema de planes, centralizar la actividad en un modelo de notificaciones reactivo, y automatizar la creación de chats grupales para coordinar dichos planes. También se incluirá la gestión avanzada de la lista de amigos.

## 2. Requisitos Técnicos
- **Framework REST:** Express.js
- **Base de Datos:** MongoDB + Mongoose
- **Tiempo Real:** Socket.io (notificaciones y eventos de planes)
- **Integración:** Triggers automáticos para creación de chats y notificaciones encapsulados en `notificationService`.

## 3. Esquemas de Base de Datos (MongoDB)

### 3.1 Esquema de Plan
- **Colección:** `plans`
- **Campos:**
  - `creator`: ObjectId (ref: User, required)
  - `title`: String (required)
  - `description`: String
  - `emojiIcon`: String (default: "📅" o "✨")
  - `datetime`: Date (required)
  - `location`: String (required)
  - `attendees`: Array de ObjectIds (ref: User)
  - `reactions`: Map de Strings (emojis) a Arrays de ObjectIds (ref: User)
  - `comments`: Array de subdocumentos:
    - `user`: ObjectId (ref: User)
    - `text`: String
    - `createdAt`: Date
  - `chatId`: ObjectId (ref: Chat) - Vinculación con el chat de grupo.
  - `createdAt`: Date

### 3.2 Esquema de Notificación
- **Colección:** `notifications`
- **Campos:**
  - `recipient`: ObjectId (ref: User, destino)
  - `sender`: ObjectId (ref: User, origen de la acción)
  - `type`: String (enum: ['plan_invite', 'plan_join', 'plan_comment', 'plan_interest', 'friend_request', 'friend_accept', 'system'])
  - `message`: String
  - `referenceId`: ObjectId (ref: Plan, User o Chat)
  - `read`: Boolean (default: false)
  - `createdAt`: Date

## 4. Endpoints Implementados

### 4.1 Planes (`/api/plans`)

#### GET /api/plans
- **Descripción:** Feed de planes propuestos.
- **Seguridad:** Autenticado mediante JWT.

#### POST /api/plans
- **Descripción:** Crear propuesta.
- **Lógica Automática:** 
  1. Crea un `Chat` de tipo `group` con el nombre del plan.
  2. Asocia el `chatId` al plan.
  3. Añade al creador como primer participante del chat y como asistente del plan.
  4. Genera notificación de `plan_interest` para usuarios con perfiles cuyos intereses coincidan.

#### POST /api/plans/:id/join
- **Descripción:** Unirse a un plan o desapuntarse (Toggle).
- **Lógica Automática:** Añade o elimina al usuario del array `attendees` del plan y del `Chat` grupal asociado, enviando notificación persistente y por socket a los interesados.

#### POST /api/plans/:id/react
- **Descripción:** Gestionar reacciones con emojis (Toggle).

#### POST /api/plans/:id/comments
- **Descripción:** Añadir comentario al hilo del plan. Envía notificaciones al creador y asistentes.

#### GET /api/plans/:id/comments
- **Descripción:** Obtener el hilo de comentarios de un plan poblado con nombres y avatares.

### 4.2 Notificaciones (`/api/notifications`)

#### GET /api/notifications
- **Descripción:** Listado cronológico de avisos (hasta 50).

#### GET /api/notifications/unread-count
- **Descripción:** Obtener el número de notificaciones no leídas para el badge.

#### PUT /api/notifications/read-all
- **Descripción:** Marcar todas las notificaciones pendientes del usuario como leídas.

#### PUT /api/notifications/:id/read
- **Descripción:** Marcar una notificación individual como leída.

### 4.3 Amigos (`/api/friends`)

#### DELETE /api/friends/:friendId
- **Descripción:** Eliminar una relación de amistad existente.

## 5. Eventos Socket.io
- `plan_join` (Emite): Al creador y al resto de participantes de un plan cuando alguien se une a un plan.
- `plan_comment` (Emite): Al creador y participantes cuando hay nuevo comentario.
- `new_notification` (Emite): Al usuario destino siempre que se cree un registro en la DB de `notifications`.

## 6. Estrategia de Pruebas (TDD)
- **Integración de Planes:** `tests/plan.test.js` y `tests/plan_reactions.test.js` (Creación -> Chat auto-generado, toggle join/react).
- **Integración de Comentarios y Sockets:** `tests/plan_comments.test.js` y `tests/socket_plans.test.js`.
- **Notificaciones:** `tests/notifications.test.js` (Intereses, comentarios, uniones, lectura de notificaciones).
- **Lógica de Amigos:** `tests/friends.test.js` (Eliminación y visibilidad).

## 7. Definición de Hecho (Definition of Done)
- [x] Modelos `Plan` y `Notification` implementados.
- [x] Lógica de creación de chat grupal automático validada.
- [x] Endpoints de unión, reacción y comentarios operativos.
- [x] Emisión de eventos socket y persistencia para notificaciones.
- [x] Tests de integración pasando al 100%.
- [x] Documentación actualizada (Swagger).

*La documentación refleja la realidad del código al cierre del Sprint 4.*