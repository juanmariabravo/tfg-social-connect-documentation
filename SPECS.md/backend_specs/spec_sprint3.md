# Especificación Técnica: Sprint 3 - Backend: Mensajería y Amigos
**Fechas:** 18/05/2026 - 31/05/2026
**Estado:** Finalizado ✅

---

## 1. Alcance del Sprint
El objetivo es proveer los endpoints y la infraestructura en tiempo real (WebSockets) para el sistema de chat y notificaciones, así como la gestión básica de amistades, estableciendo la base social de la aplicación.

## 2. Requisitos Técnicos
- **Framework REST:** Express.js
- **Base de Datos:** MongoDB + Mongoose
- **Tiempo Real:** Socket.io (para eventos síncronos)
- **Autenticación en Sockets:** Middleware para validar JWT en la conexión de WebSockets.

## 3. Esquemas de Base de Datos (MongoDB)

### 3.1 Esquema de Chat
- **Colección:** `chats`
- **Campos:**
  - `isGroup` (Boolean, default: false)
  - `name` (String, opcional para grupos)
  - `emojiIcon` (String, opcional para grupos)
  - `participants` (Array de ObjectIds, ref: User)
  - `lastMessage` (ObjectId, ref: Message)
  - `createdAt` (Date)
  - `updatedAt` (Date)

### 3.2 Esquema de Message
- **Colección:** `messages`
- **Campos:**
  - `chatId` (ObjectId, ref: Chat)
  - `sender` (ObjectId, ref: User)
  - `content` (String, required)
  - `readBy` (Array de ObjectIds, ref: User)
  - `createdAt` (Date)

### 3.3 Esquema de Amistad
- **Colección:** `friendships`
- **Campos:**
  - `requester` (ObjectId, ref: User) - Quien envía la solicitud.
  - `receiver` (ObjectId, ref: User) - Quien la recibe.
  - `status` (String, enum: ['pending', 'accepted', 'rejected'])
  - `createdAt` (Date)
  - `updatedAt` (Date)

## 4. Endpoints Implementados

### 4.1 Mensajería (Chats)

#### GET /api/chats
- **Descripción:** Listar los chats del usuario logueado, ordenados por fecha de actualización.
- **Funcionalidad Extra:** Incluye `unreadCount` calculado dinámicamente para cada chat.
- **Autenticación:** Required (JWT).

#### POST /api/chats
- **Descripción:** Crear un chat individual o grupal. Si ya existe un chat individual entre dos personas, devuelve el existente.
- **Cuerpo:** `{ "participants": ["userId1"], "isGroup": boolean, "name": "Nombre", "emojiIcon": "🚀" }`

#### GET /api/chats/:chatId/messages
- **Descripción:** Obtener el historial de mensajes de un chat con **paginación por cursor**.
- **Query Params:** `cursor` (ID del mensaje), `limit` (default: 20).
- **Seguridad:** Valida que el usuario pertenece al chat.

#### POST /api/chats/:chatId/messages
- **Descripción:** Enviar un mensaje. Actualiza el `lastMessage` del chat y emite el evento `new_message`.

### 4.2 Gestión de Amigos y Solicitudes

#### POST /api/friends/request/:userId
- **Descripción:** Enviar una solicitud de amistad. Evita duplicados y auto-solicitudes.

#### GET /api/friends/requests
- **Descripción:** Listar solicitudes recibidas y pendientes.

#### PUT /api/friends/request/:requestId
- **Descripción:** Aceptar o rechazar una solicitud.

#### GET /api/friends
- **Descripción:** Listar amigos confirmados con sus perfiles populados.

## 5. Notificaciones Síncronas (WebSockets)
- **Tecnología:** Socket.io con autenticación vía Handshake (JWT).
- **Eventos Definidos (Sincronizados con Frontend):**
  - `user_status_change` (Emite): Notifica a todos cuando un usuario se conecta (online) o desconecta (offline).
  - `new_message` (Emite): Envía el contenido del mensaje y metadatos a los participantes del chat.
  - `join_chat` (Recibe): El cliente notifica que ha entrado en una sala; el servidor marca mensajes como leídos.
  - `leave_chat` (Recibe): El cliente notifica que sale de la sala de chat.
  - `friend_request` (Preparado): Infraestructura lista para notificar solicitudes de amistad.
- **Seguridad:** Middleware que valida el JWT antes de permitir la conexión al socket.

## 6. Estrategia de Pruebas (TDD)
- **Tests de Seguridad:** Verificado que un usuario ajeno no puede acceder a los mensajes de un chat.
- **Tests de Integración:** Verificada la creación de chats y envío de mensajes REST -> Socket.

## 7. Definición de Hecho (Definition of Done)
- [x] Modelos de datos para Chat, Message y Friendship integrados en Mongoose.
- [x] Endpoints REST de chats y mensajes con paginación por cursor.
- [x] Lógica de `unreadCount` implementada en el listado de chats.
- [x] Test de seguridad de pertenencia a chat implementado y pasando.
- [x] Endpoint de lista de amigos y gestión de solicitudes funcional.
- [x] Servidor Socket.io configurado con validación de JWT.
- [x] Gestión de estados Online/Offline mediante eventos globales de Socket.
- [x] Código mergeado en `main`.

*La documentación refleja la realidad del código al cierre del Sprint 3.*