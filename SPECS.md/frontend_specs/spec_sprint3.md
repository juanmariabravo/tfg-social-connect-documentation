# Especificación Técnica: Sprint 3 - Frontend: Mensajería y Navegación
**Fechas:** 18/05/2026 - 31/05/2026
**Estado:** Finalizado ✅

---

## 0. Fase de Diseño (Mockups)
**Herramienta:** Lovable
**Objetivo:** Diseñar la experiencia de usuario para la comunicación y navegación.

### 0.1 Mockups Implementados
- [x] **Navbar:** Barra de navegación global adaptada. Incluye indicadores de navegación activa.
- [x] **Chat (Listado):** Vista de conversaciones con avatares, últimos mensajes, fechas relativas y contadores de mensajes no leídos (`unreadCount`).
- [x] **Chat (Ventana):** Interfaz moderna con burbujas, soporte para grupos (nombres de remitentes), scroll automático al recibir y scroll infinito hacia arriba.
- [x] **Nuevo Chat:** Modal avanzado con selección de tipo (Individual o Grupo), selector de emojis e input de nombre de grupo.

---

## 1. Alcance del Sprint
Construir la estructura global de navegación (Navbar), desarrollar todo el sistema de UI para la mensajería y establecer la conexión por WebSockets para manejar notificaciones y mensajes en tiempo real.

## 2. Requisitos Técnicos
- **Framework:** React 19 (TypeScript)
- **Estilos:** Tailwind CSS 4 + Componentes UI (Lucide React, Radix/shadcn).
- **WebSockets:** `socket.io-client`.
- **Gestión de Estado Síncrono:** Context API (`SocketContext.tsx`).
- **Navegación:** React Router 7.

## 3. Vistas y Componentes Implementados

### 3.1 Navbar (Navegación Global)
- **Funcionalidad:** Navegación entre Inicio, Explorar, Planes, Chats, Perfil.
- **Notificaciones Síncronas:** Alertas visuales reactivas al recibir eventos de socket.

### 3.2 Chat - Listado (/chat)
- **Funcionalidad:**
  - Consumo de `GET /api/chats`.
  - Filtro de búsqueda por nombre de chat/usuario.
  - Indicadores de mensajes no leídos en tiempo real.
  - Formato de fechas relativo (ej: "hace 2 min").

### 3.3 Chat - Nuevo (Modal)
- **Funcionalidad:**
  - Selección de modo: Chat Individual o Crear Grupo.
  - Selección múltiple de amigos para grupos.
  - Personalización de grupo con Nombre y Emoji (integración con `EmojiPicker`).
  - Redirección automática al nuevo chat creado.

### 3.4 Chat - Ventana de Conversación (/chat/:chatId)
- **Funcionalidad:**
  - **Scroll Infinito:** Carga de mensajes antiguos al llegar al tope superior mediante paginación por cursor.
  - **Auto-scroll:** Desplazamiento inteligente al final al enviar o recibir mensajes.
  - **Status:** Indicador de estado del otro usuario (Activo ahora / Desconectado) basado en eventos de socket.
  - **Input:** Envío con `Enter`, soporte para emojis y estado de "Enviando...".

## 4. Integración Síncrona (WebSockets)
- **Tecnología:** `socket.io-client` con autenticación por JWT.
- **Eventos Definidos (Sincronizados con Backend):**
  - `user_status_change` (Escucha): Actualiza globalmente el estado de conexión de los usuarios (Online/Offline).
  - `new_message` (Escucha): Recibe mensajes en tiempo real para actualizar la ventana de chat y el `unreadCount`.
  - `join_chat` (Emite): Informa al servidor que el usuario está viendo un chat específico.
  - `leave_chat` (Emite): Informa al servidor que el usuario ha dejado de ver el chat.
  - `friend_request` (Escucha): Preparado para mostrar notificaciones de nuevas solicitudes.
- **Gestión de Estado:** Implementado mediante `SocketProvider` para disponibilidad global del socket y estado de conexión.

## 5. Estructura de Componentes Clave
- `src/context/SocketContext.tsx`: Corazón de la comunicación en tiempo real.
- `src/pages/Chat.tsx`: Vista principal unificada (Sidebar + Ventana).
- `src/components/NewChatModal.tsx`: Lógica compleja de creación de hilos sociales.
- `src/components/ui/emoji-picker.tsx`: Componente reutilizable para selección de emojis.

## 6. Definición de Hecho (Definition of Done)
- [x] Navbar funcional con indicador de notificaciones reactivo a WebSockets.
- [x] Listado de chats conectado al backend con soporte para mensajes no leídos.
- [x] Modal de nuevo chat con soporte para grupos y personalización.
- [x] Ventana de chat operativa con scroll infinito y auto-scroll.
- [x] Conexión de `socket.io-client` global y autenticada.
- [x] Indicadores de estado Online/Offline funcionando en tiempo real.
- [x] Integración de EmojiPicker en el chat y creación de grupos.
- [x] Código verificado y mergeado en `main`.

*La documentación refleja la realidad del código al cierre del Sprint 3.*