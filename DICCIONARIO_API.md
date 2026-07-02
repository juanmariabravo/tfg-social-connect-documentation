# Diccionario de la API Completo - Social Connect

Este documento recopila la documentación técnica exhaustiva y actualizada del backend de **Social Connect**, detallando la arquitectura, la autenticación, los modelos de datos, el algoritmo de afinidad (**WCScore**), todos los endpoints REST (enrutados bajo `/api`) y el protocolo de comunicación en tiempo real mediante **WebSockets** (Socket.io).

---

## Índice

1. [Arquitectura y Configuración General](#1-arquitectura-y-configuración-general)
2. [Autenticación y Seguridad](#2-autenticación-y-seguridad)
3. [Modelos de Datos (Mongoose/MongoDB)](#3-modelos-de-datos-mongoosemongodb)
4. [Algoritmo de Compatibilidad (Weighted Compatibility Score - WCScore)](#4-algoritmo-de-compatibilidad-weighted-compatibility-score---wcscore)
5. [Diccionario de Endpoints REST (API HTTP)](#5-diccionario-de-endpoints-rest-api-http)
   - [5.1 Estado del Servidor (Status)](#51-estado-del-servidor-status)
   - [5.2 Autenticación (Auth)](#52-autenticación-auth)
   - [5.3 Perfiles de Usuario (Profiles)](#53-perfiles-de-usuario-profiles)
   - [5.4 Amigos y Solicitudes (Friends)](#54-amigos-y-solicitudes-friends)
   - [5.5 Chats y Mensajería (Chats)](#55-chats-y-mensajería-chats)
   - [5.6 Planes y Eventos (Plans)](#56-planes-y-eventos-plans)
   - [5.7 Notificaciones (Notifications)](#57-notificaciones-notifications)
6. [Comunicación en Tiempo Real (WebSockets / Socket.io)](#6-comunicación-en-tiempo-real-websockets--socketio)

---

## 1. Arquitectura y Configuración General

El backend está desarrollado sobre el entorno **Node.js** utilizando el framework web **Express.js** bajo el patrón **Modelo-Vista-Controlador (MVC)**. La persistencia de datos se gestiona con **MongoDB** mediante el ODM **Mongoose**.

- **Puerto de desarrollo:** `5000` (configurable mediante la variable de entorno `PORT`).
- **Punto de acceso API REST:** `/api`
- **Límite de tamaño de petición:** `50mb` (configurado en Express para soportar imágenes codificadas en Base64).
- **Documentación Interactiva (Swagger):** Disponible en la ruta `/docs` utilizando `swagger-ui-express` y `swagger-jsdoc`.
- **Comunicación síncrona/tiempo real:** Implementada mediante **Socket.io**.

---

## 2. Autenticación y Seguridad

La API emplea un mecanismo de autenticación sin estado basado en **JSON Web Tokens (JWT)**.

### Flujo de Autenticación REST
1. El cliente envía credenciales mediante `/api/auth/register` o `/api/auth/login`.
2. El servidor responde con un token JWT firmado.
3. El cliente debe almacenar este token e incluirlo en la cabecera HTTP de todas las peticiones a rutas protegidas mediante el esquema **Bearer**:
   ```http
   Authorization: Bearer <TOKEN_JWT>
   ```
4. El middleware `authMiddleware` extrae el token, verifica su validez y adjunta la información decodificada del usuario a la petición (`req.user`).

### Autenticación en WebSockets (Socket.io)
Antes de establecer la conexión de WebSocket, el servidor exige la autenticación en el handshake:
- El token puede enviarse en el objeto de autenticación del socket (`socket.handshake.auth.token`) o en las cabeceras tradicionales (`socket.handshake.headers['authorization']`).
- Si el token no se proporciona o es inválido, la conexión de red se rechaza inmediatamente con un error de autenticación.

---

## 3. Modelos de Datos (Mongoose/MongoDB)

### 3.1 Usuario (`User`)
Almacena las credenciales de inicio de sesión e información de identidad base del usuario.

| Campo | Tipo | Requerido | Descripción / Validación |
| :--- | :--- | :--- | :--- |
| `_id` | `ObjectId` | Auto | Identificador único de MongoDB. |
| `username` | `String` | Sí | Nombre de usuario único para mostrar. |
| `email` | `String` | Sí | Dirección de correo única, convertida a minúsculas, validada con expresión regular. |
| `password` | `String` | Sí | Contraseña encriptada con `bcryptjs`. |
| `dateOfBirth` | `String` | Sí | Fecha de nacimiento en formato ISO string (`YYYY-MM-DD`). |
| `createdAt` | `Date` | Auto | Fecha de creación del registro. |
| `updatedAt` | `Date` | Auto | Fecha de última modificación. |

**Relaciones Virtuales:**
- `profile`: Referencia de uno a uno al modelo `Profile` buscando `Profile.userId === User._id`.

---

### 3.2 Perfil de Usuario (`Profile`)
Contiene los datos detallados de configuración del perfil del usuario, intereses, geolocalización y rasgos de personalidad.

| Campo | Tipo | Requerido | Descripción / Validación |
| :--- | :--- | :--- | :--- |
| `userId` | `ObjectId` | Sí | Referencia indexada y única al modelo `User`. |
| `bio` | `String` | No | Biografía o descripción del usuario. |
| `location` | `String` | No | Dirección o nombre de la ubicación textual. |
| `avatar` | `String` | No | URL de imagen o avatar virtual predefinido. |
| `photo` | `String` | No | URL o String Base64 de la foto real del usuario. |
| `coordinates` | `Object` | No | Coordenadas espaciales `{ lat: Number, lng: Number }`. |
| `interests` | `Array` | No | Lista de intereses. Cada elemento es `{ name: String, emoji: String }`. |
| `personality` | `Object` | No | Rasgos de personalidad Big Five (valores de 1 a 10):<br>- `extroversion` (Número)<br>- `openness` (Número)<br>- `conscientiousness` (Número)<br>- `agreeableness` (Número)<br>- `neuroticism` (Número) |
| `onboardingCompleted` | `Boolean`| Sí | Indica si el usuario finalizó la configuración inicial (Default: `false`). |

---

### 3.3 Amistad (`Friendship`)
Gestiona el estado social bidireccional entre dos usuarios.

| Campo | Tipo | Requerido | Descripción / Validación |
| :--- | :--- | :--- | :--- |
| `requester` | `ObjectId` | Sí | Referencia al usuario `User` que envía la solicitud. |
| `receiver` | `ObjectId` | Sí | Referencia al usuario `User` que recibe la solicitud. |
| `status` | `String` | Sí | Estado de la relación: `['pending', 'accepted', 'rejected']` (Default: `'pending'`). |

**Restricciones de índice:**
- Índice único compuesto sobre `{ requester: 1, receiver: 1 }` para evitar duplicar solicitudes.

---

### 3.4 Chat (`Chat`)
Almacena las salas de chat individuales o grupales.

| Campo | Tipo | Requerido | Descripción / Validación |
| :--- | :--- | :--- | :--- |
| `isGroup` | `Boolean` | Sí | `true` si es un chat grupal (ej: asociado a un Plan), `false` si es individual (Default: `false`). |
| `name` | `String` | No | Nombre del grupo (requerido para grupos). |
| `emojiIcon` | `String` | No | Icono emoji para identificar el grupo visualmente. |
| `participants` | `[ObjectId]`| Sí | Array de referencias a `User`. Debe tener al menos 1 participante. |
| `lastMessage` | `ObjectId` | No | Referencia al último mensaje (`Message`) enviado para optimización de consultas. |

---

### 3.5 Mensaje (`Message`)
Representa los mensajes individuales enviados dentro de una sala de chat.

| Campo | Tipo | Requerido | Descripción / Validación |
| :--- | :--- | :--- | :--- |
| `chatId` | `ObjectId` | Sí | Referencia al `Chat` contenedor. |
| `sender` | `ObjectId` | Sí | Referencia al `User` autor. |
| `content` | `String` | Sí | Contenido de texto del mensaje. |
| `readBy` | `[ObjectId]`| Sí | Array de referencias a `User` que ya han visualizado el mensaje. |

---

### 3.6 Plan (`Plan`)
Define propuestas de eventos sociales que sirven como "excusa para hablar".

| Campo | Tipo | Requerido | Descripción / Validación |
| :--- | :--- | :--- | :--- |
| `creator` | `ObjectId` | Sí | Referencia al `User` organizador. |
| `title` | `String` | Sí | Título del plan propuesto. |
| `description` | `String` | Sí | Descripción detallada de la actividad. |
| `emojiIcon` | `String` | No | Emoji distintivo del plan (ej. "🍕", "🎬"). |
| `datetime` | `Date` | Sí | Fecha y hora de realización del evento. |
| `location` | `String` | Sí | Lugar físico donde se llevará a cabo. |
| `attendees` | `[ObjectId]`| Sí | Array de referencias a `User` que asisten al plan. |
| `chatId` | `ObjectId` | No | Referencia al `Chat` grupal autogenerado para coordinar el plan. |
| `reactions` | `Map` | Sí | Mapa de String (Emoji) a Array de `ObjectId` (`User`) que reaccionaron. |
| `comments` | `[Object]` | No | Array de subdocumentos de comentarios:<br>- `user` (`ObjectId`, ref: `User`, req)<br>- `text` (`String`, req)<br>- `createdAt` (`Date`, default: `Date.now`) |

---

### 3.7 Notificación (`Notification`)
Almacena avisos persistentes destinados a captar la atención de los usuarios.

| Campo | Tipo | Requerido | Descripción / Validación |
| :--- | :--- | :--- | :--- |
| `recipient` | `ObjectId` | Sí | Referencia al `User` receptor de la notificación. |
| `sender` | `ObjectId` | No | Referencia al `User` causante de la notificación. |
| `type` | `String` | Sí | Tipo de notificación: `['plan_invite', 'plan_join', 'plan_comment', 'plan_interest', 'friend_request', 'friend_accept', 'system']`. |
| `message` | `String` | Sí | Mensaje de texto descriptivo estructurado en castellano. |
| `referenceId` | `ObjectId` | No | ID de referencia del recurso implicado (un `Plan`, `User` o `Chat`). |
| `read` | `Boolean` | Sí | Estado de lectura (Default: `false`). |

---

## 4. Algoritmo de Compatibilidad (Weighted Compatibility Score - WCScore)

El motor de emparejamiento de **Social Connect** calcula un porcentaje de afinidad de **0% a 100%** entre dos perfiles utilizando el algoritmo **Weighted Compatibility Score (WCScore)**. La fórmula ponderada asigna importancia relativa a cuatro pilares:

$$\text{WCScore} = 0.35 \cdot S_{\text{personalidad}} + 0.20 \cdot S_{\text{intereses}} + 0.30 \cdot S_{\text{proximidad}} + 0.15 \cdot S_{\text{edad}}$$

### 4.1 Personalidad (35% - Modelo Big Five)
Calcula la cercanía de los rasgos de personalidad utilizando la **distancia euclidiana** en un espacio de 5 dimensiones. Cada rasgo toma un valor de 1 a 10.
- **Fórmula matemática:**
  $$S_{\text{personalidad}} = \max\left(0, \, 100 \times \left(1 - \frac{\sqrt{\sum_{i=1}^{5} (A_i - B_i)^2}}{\sqrt{5 \times 9^2}}\right)\right)$$
- *Donde $A_i$ y $B_i$ son los rasgos correspondientes a cada usuario, y $\sqrt{5 \times 9^2} \approx 20.124$ es la máxima distancia teórica posible entre dos perfiles opuestos.*

### 4.2 Intereses Comunes (20% - Índice de Jaccard)
Mide la similitud entre los conjuntos de intereses (ignorando mayúsculas/minúsculas) de ambos usuarios.
- **Fórmula matemática:**
  $$S_{\text{intereses}} = \frac{|A \cap B|}{|A \cup B|} \times 100$$
- *Si ninguno de los dos usuarios ha seleccionado intereses en su perfil, se retorna un score de compatibilidad por intereses por defecto de 100%.*

### 4.3 Proximidad Geográfica (30% - Distancia Haversine)
Determina la distancia física en kilómetros a partir de la latitud y longitud de ambos perfiles utilizando la **fórmula de Haversine**.
- **Fórmula de Haversine:**
  $$d = 2 R \arcsin\left(\sqrt{\sin^2\left(\frac{\Delta\text{lat}}{2}\right) + \cos(\text{lat}_1)\cos(\text{lat}_2)\sin^2\left(\frac{\Delta\text{lng}}{2}\right)}\right)$$
- **Conversión a Score:**
  $$S_{\text{proximidad}} = \begin{cases} 100, & \text{si } d \le 5 \text{ km} \\ \max\left(0, \, 100 \times \left(1 - \frac{d - 5}{95}\right)\right), & \text{si } d > 5 \text{ km} \end{cases}$$
- *Esto garantiza un 100% de afinidad para distancias menores o iguales a 5 km, y decae linealmente hasta un 0% cuando la distancia alcanza o supera los 100 km.*

### 4.4 Similitud de Edad (15%)
Calcula la diferencia absoluta de edades basándose en las fechas de nacimiento.
- **Fórmula matemática:**
  $$S_{\text{edad}} = \max\left(0, \, 100 \times \left(1 - \frac{|\text{Edad}_A - \text{Edad}_B|}{20}\right)\right)$$
- *Esto otorga un 100% si tienen la misma edad, disminuyendo progresivamente hasta alcanzar el 0% al haber una diferencia de 20 o más años.*

---

## 5. Diccionario de Endpoints REST (API HTTP)

Todas las respuestas con éxito devuelven un código de estado `2xx`. Las peticiones fallidas devuelven códigos `4xx` o `5xx` acompañados de un objeto de error descriptivo:
```json
{
  "message": "Mensaje descriptivo del error"
}
```

### 5.1 Estado del Servidor (Status)

#### `GET /api/status`
- **Descripción:** Comprueba que el servidor Express y la base de datos MongoDB estén operativos de forma correcta.
- **Requiere Autenticación:** No.
- **Respuesta de Éxito (`200 OK`):**
  ```json
  {
    "status": "online",
    "timestamp": "2026-07-01T12:00:00.000Z",
    "database": "connected"
  }
  ```

---

### 5.2 Autenticación (Auth)

#### `POST /api/auth/register`
- **Descripción:** Registra una nueva cuenta de usuario en la aplicación y crea de forma automática un perfil vacío vinculado.
- **Requiere Autenticación:** No.
- **Cuerpo de la Petición (`application/json`):**
  ```json
  {
    "username": "juan_perez",
    "email": "juan@example.com",
    "password": "mi_password_segura",
    "dateOfBirth": "1998-05-15"
  }
  ```
- **Respuesta de Éxito (`201 Created`):**
  ```json
  {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "60b8d2f5f3b3b211a8c1f9b1",
      "username": "juan_perez",
      "email": "juan@example.com",
      "dateOfBirth": "1998-05-15"
    }
  }
  ```
- **Respuestas de Error:**
  - `400 Bad Request`: Datos de validación inválidos o el correo ya está registrado en el sistema.

#### `POST /api/auth/login`
- **Descripción:** Inicia la sesión de un usuario registrado validando sus credenciales de acceso.
- **Requiere Autenticación:** No.
- **Cuerpo de la Petición (`application/json`):**
  ```json
  {
    "email": "juan@example.com",
    "password": "mi_password_segura"
  }
  ```
- **Respuesta de Éxito (`200 OK`):**
  ```json
  {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "60b8d2f5f3b3b211a8c1f9b1",
      "username": "juan_perez",
      "email": "juan@example.com"
    }
  }
  ```
- **Respuestas de Error:**
  - `400 Bad Request`: Email/password omitido o credenciales incorrectas.

#### `GET /api/auth/me`
- **Descripción:** Devuelve la información del usuario autenticado en el token actual de sesión.
- **Requiere Autenticación:** Sí (JWT).
- **Respuesta de Éxito (`200 OK`):**
  ```json
  {
    "_id": "60b8d2f5f3b3b211a8c1f9b1",
    "username": "juan_perez",
    "email": "juan@example.com",
    "dateOfBirth": "1998-05-15",
    "createdAt": "2026-05-10T15:20:00.000Z"
  }
  ```
- **Respuestas de Error:**
  - `401 Unauthorized`: Token no provisto o inválido.
  - `404 Not Found`: El usuario ya no existe en la base de datos.

---

### 5.3 Perfiles de Usuario (Profiles)

#### `GET /api/profiles/explore`
- **Descripción:** Busca perfiles recomendados de otros usuarios excluyendo a los que ya son amigos o con solicitudes de amistad en trámite. Los ordena por compatibilidad o rasgos específicos.
- **Requiere Autenticación:** Sí (JWT).
- **Parámetros de Consulta (Query):**
  - `filter` (Opcional): Estrategia de ordenación principal. Valores admitidos:
    - `all` (Por defecto): Ordena de mayor a menor según el **WCScore** global.
    - `proximity`: Prioriza menor distancia geográfica.
    - `interests`: Prioriza mayor cantidad de intereses comunes.
    - `personality`: Prioriza similitud de rasgos Big Five.
    - `age`: Prioriza menor diferencia de edad.
  - `search` (Opcional): Cadena de texto para buscar por nombre de usuario o nombre de interés.
  - `page` (Opcional, Default: `1`): Número de página para la paginación.
  - `limit` (Opcional, Default: `10`): Resultados por página.
- **Respuesta de Éxito (`200 OK`):**
  ```json
  {
    "profiles": [
      {
        "_id": "60b8d2f5f3b3b211a8c1f9b5",
        "userId": {
          "_id": "60b8d2f5f3b3b211a8c1f9c0",
          "username": "sofia_gomez",
          "email": "sofia@example.com",
          "dateOfBirth": "1999-09-20"
        },
        "bio": "Apasionada del senderismo y la fotografía",
        "location": "Albacete, España",
        "avatar": "avatar_girl_3.png",
        "photo": "https://img...",
        "interests": [
          { "name": "Senderismo", "emoji": "🥾" },
          { "name": "Fotografía", "emoji": "📷" }
        ],
        "personality": {
          "extroversion": 8,
          "openness": 9,
          "conscientiousness": 7,
          "agreeableness": 8,
          "neuroticism": 3
        },
        "compatibility": {
          "personality": 88,
          "interests": 50,
          "proximity": 98,
          "age": 95,
          "total": 84
        }
      }
    ],
    "pagination": {
      "total": 12,
      "page": 1,
      "pages": 2,
      "limit": 10
    }
  }
  ```

#### `GET /api/profiles/reverse-geocode`
- **Descripción:** Obtiene una representación textual legible de una dirección/municipio a partir de unas coordenadas geográficas (`lat`, `lng`).
- **Requiere Autenticación:** Sí (JWT).
- **Parámetros de Consulta (Query):**
  - `lat` (Requerido): Latitud.
  - `lng` (Requerido): Longitud.
- **Respuesta de Éxito (`200 OK`):**
  ```json
  {
    "address": "Ciudad Real, Castilla-La Mancha, España"
  }
  ```

#### `GET /api/profiles/random`
- **Descripción:** Obtiene perfiles aleatorios sin aplicar filtros de afinidad pesados. Útil para pantallas rápidas de descubrimiento.
- **Requiere Autenticación:** Sí (JWT).
- **Parámetros de Consulta (Query):**
  - `limit` (Opcional, Default: `5`): Cantidad de perfiles a obtener.
- **Respuesta de Éxito (`200 OK`):**
  ```json
  [
    {
      "_id": "60b8d2f5f3b3b211a8c1f9b8",
      "userId": { "_id": "60b8f...", "username": "carlos_88" },
      "bio": "Melómano empedernido",
      "location": "Toledo",
      "avatar": "avatar_man_1.png",
      "interests": [{ "name": "Música", "emoji": "🎵" }]
    }
  ]
  ```

#### `GET /api/profiles/:userId`
- **Descripción:** Obtiene la ficha de perfil de un usuario concreto a través de su identificador de usuario.
- **Requiere Autenticación:** Sí (JWT).
- **Parámetros de Ruta:**
  - `userId` (Requerido): Identificador único (`ObjectId`) del usuario.
- **Respuesta de Éxito (`200 OK`):**
  ```json
  {
    "_id": "60b8d2f5f3b3b211a8c1f9b2",
    "userId": {
      "_id": "60b8d2f5f3b3b211a8c1f9b1",
      "username": "juan_perez",
      "email": "juan@example.com",
      "dateOfBirth": "1998-05-15"
    },
    "bio": "Estudiante de Ingeniería",
    "location": "Ciudad Real, España",
    "avatar": "avatar_man_3.png",
    "photo": "https://...",
    "interests": [
      { "name": "Cine", "emoji": "🎬" }
    ],
    "personality": {
      "extroversion": 5,
      "openness": 7,
      "conscientiousness": 6,
      "agreeableness": 7,
      "neuroticism": 5
    },
    "onboardingCompleted": true
  }
  ```
- **Respuestas de Error:**
  - `404 Not Found`: Si el formato del ID es correcto pero no existe el perfil de ese usuario.

#### `PUT /api/profiles/:userId`
- **Descripción:** Modifica los campos configurables del perfil del usuario logueado. Si el campo opcional `username` es provisto en la petición, actualiza también de forma síncrona el nombre de usuario asociado en el modelo base `User`.
- **Requiere Autenticación:** Sí (JWT).
- **Parámetros de Ruta:**
  - `userId`: Identificador del usuario. Debe coincidir con el usuario autenticado (ID extraído del token JWT).
- **Cuerpo de la Petición (`application/json`):**
  ```json
  {
    "username": "juan_perez_editado",
    "bio": "Nueva biografía genial.",
    "location": "Ciudad Real, España",
    "coordinates": { "lat": 38.98626, "lng": -3.92754 },
    "avatar": "avatar_man_4.png",
    "photo": "data:image/png;base64,iVBORw0KG...",
    "interests": [
      { "name": "Cine", "emoji": "🎬" },
      { "name": "Programación", "emoji": "💻" }
    ],
    "personality": {
      "extroversion": 6,
      "openness": 8,
      "conscientiousness": 6,
      "agreeableness": 8,
      "neuroticism": 4
    },
    "onboardingCompleted": true
  }
  ```
- **Respuesta de Éxito (`200 OK`):** Retorna el documento de `Profile` modificado con los nuevos datos actualizados.
- **Respuestas de Error:**
  - `403 Forbidden`: Intento de modificar un perfil ajeno (`req.user.userId !== req.params.userId`).
  - `404 Not Found`: Perfil no localizado.

---

### 5.4 Amigos y Solicitudes (Friends)

#### `POST /api/friends/request/:userId`
- **Descripción:** Envía una solicitud de amistad dirigida al usuario indicado por `userId`. Valida que no exista relación previa.
- **Requiere Autenticación:** Sí (JWT).
- **Parámetros de Ruta:**
  - `userId`: Identificador del usuario destinatario de la solicitud.
- **Respuesta de Éxito (`201 Created`):**
  ```json
  {
    "_id": "60b8d2f5f3b3b211a8c1fa01",
    "requester": "60b8d2f5f3b3b211a8c1f9b1",
    "receiver": "60b8d2f5f3b3b211a8c1f9b5",
    "status": "pending",
    "createdAt": "2026-07-01T12:05:00.000Z",
    "updatedAt": "2026-07-01T12:05:00.000Z"
  }
  ```
- **Lógica asociada:**
  - Crea un registro de tipo `friend_request` en el modelo `Notification`.
  - Envía un evento WebSocket instantáneo `new_notification` al usuario destinatario de la solicitud.
- **Respuestas de Error:**
  - `400 Bad Request`: Auto-solicitud (enviarse a uno mismo) o si ya existe una solicitud activa/amistad entre ambos.

#### `GET /api/friends/requests`
- **Descripción:** Recupera la lista de solicitudes de amistad recibidas y que se encuentran en estado pendiente para el usuario autenticado.
- **Requiere Autenticación:** Sí (JWT).
- **Respuesta de Éxito (`200 OK`):**
  ```json
  [
    {
      "_id": "60b8d2f5f3b3b211a8c1fa01",
      "requester": {
        "_id": "60b8d2f5f3b3b211a8c1f9b1",
        "username": "juan_perez",
        "profile": {
          "avatar": "avatar_man_3.png",
          "location": "Ciudad Real"
        }
      },
      "status": "pending",
      "createdAt": "2026-07-01T12:05:00.000Z"
    }
  ]
  ```

#### `PUT /api/friends/request/:requestId`
- **Descripción:** Permite aceptar o rechazar una solicitud de amistad pendiente.
- **Requiere Autenticación:** Sí (JWT).
- **Parámetros de Ruta:**
  - `requestId`: Identificador único (`_id`) de la solicitud en la colección `friendships`.
- **Cuerpo de la Petición (`application/json`):**
  ```json
  {
    "status": "accepted" 
  }
  ```
  *(Valores válidos para status: `accepted` o `rejected`)*
- **Respuesta de Éxito (`200 OK`):** Retorna la relación modificada.
- **Lógica asociada (si se acepta):**
  - Genera automáticamente una notificación para el solicitante original (`friend_accept`).
  - Envía el evento WebSocket de `new_notification` al destinatario original en tiempo real.
- **Respuestas de Error:**
  - `403 Forbidden`: Intento de responder a una solicitud donde el usuario logueado no es el receptor.
  - `404 Not Found`: Solicitud no encontrada.

#### `GET /api/friends`
- **Descripción:** Obtiene la lista con los amigos confirmados del usuario (amistades con estado `'accepted'`).
- **Requiere Autenticación:** Sí (JWT).
- **Respuesta de Éxito (`200 OK`):** Devuelve un array de objetos de perfiles de usuario que han sido confirmados como amigos, incluyendo los campos del perfil y estado de conexión online actual.

#### `DELETE /api/friends/:friendId`
- **Descripción:** Rompe una relación de amistad o cancela una solicitud de amistad existente.
- **Requiere Autenticación:** Sí (JWT).
- **Parámetros de Ruta:**
  - `friendId`: Identificador del amigo.
- **Respuesta de Éxito (`200 OK`):**
  ```json
  {
    "message": "Amigo eliminado correctamente."
  }
  ```

---

### 5.5 Chats y Mensajería (Chats)

#### `GET /api/chats`
- **Descripción:** Devuelve los chats en los que participa el usuario logueado, ordenados cronológicamente por la fecha del último mensaje (`updatedAt`).
- **Requiere Autenticación:** Sí (JWT).
- **Lógica asociada:**
  - Calcula dinámicamente un contador de mensajes sin leer (`unreadCount`) para el usuario logueado comparando los mensajes del chat que no contienen el ID del usuario en su array `readBy`.
- **Respuesta de Éxito (`200 OK`):**
  ```json
  [
    {
      "_id": "60b8d2f5f3b3b211a8c1fb10",
      "isGroup": false,
      "participants": [
        { "_id": "60b8d...", "username": "juan_perez" },
        { "_id": "60b8f...", "username": "sofia_gomez" }
      ],
      "lastMessage": {
        "_id": "60b8d...",
        "content": "¡Hola! ¿Cómo estás?",
        "sender": "60b8f...",
        "createdAt": "2026-07-01T12:10:00.000Z"
      },
      "unreadCount": 1,
      "updatedAt": "2026-07-01T12:10:00.000Z"
    }
  ]
  ```

#### `POST /api/chats`
- **Descripción:** Inicia una conversación grupal o de tipo individual.
- **Requiere Autenticación:** Sí (JWT).
- **Cuerpo de la Petición (`application/json`):**
  ```json
  {
    "participants": ["60b8d2f5f3b3b211a8c1f9b5"],
    "isGroup": false,
    "name": "Grupo de Cine",
    "emojiIcon": "🎬"
  }
  ```
- **Lógica asociada:**
  - Si `isGroup === false` (individual) y ya existe un chat uno a uno previo entre ambos, el controlador detecta el duplicado y devuelve el objeto del chat existente (Código `200 OK`) en lugar de duplicarlo.
- **Respuesta de Éxito (`201 Created` / `200 OK`):** Devuelve el objeto del chat guardado o recuperado.

#### `GET /api/chats/:chatId/messages`
- **Descripción:** Obtiene los mensajes de la sala de chat. Implementado mediante **paginación clásica** (`page` y `limit`) para la carga progresiva en scroll.
- **Requiere Autenticación:** Sí (JWT).
- **Parámetros de Ruta:**
  - `chatId`: Identificador único del chat.
- **Parámetros de Consulta (Query):**
  - `page` (Opcional, Default: `1`): Página del historial de mensajes.
  - `limit` (Opcional, Default: `20`): Mensajes devueltos por página.
- **Respuesta de Éxito (`200 OK`):**
  ```json
  {
    "messages": [
      {
        "_id": "60b8d2f5f3b3b211a8c1fb55",
        "chatId": "60b8d2f5f3b3b211a8c1fb10",
        "sender": {
          "_id": "60b8d2f5f3b3b211a8c1f9b1",
          "username": "juan_perez",
          "email": "juan@example.com"
        },
        "content": "¡Hola! ¿Cómo estás?",
        "readBy": ["60b8d2f5f3b3b211a8c1f9b1", "60b8d2f5f3b3b211a8c1f9b5"],
        "createdAt": "2026-07-01T12:10:00.000Z"
      }
    ],
    "pagination": {
      "total": 45,
      "page": 1,
      "pages": 3,
      "limit": 20
    }
  }
  ```
- **Respuestas de Error:**
  - `403 Forbidden`: Si el usuario logueado no figura como participante dentro de los miembros del chat.
  - `404 Not Found`: Chat inexistente.

#### `POST /api/chats/:chatId/messages`
- **Descripción:** Envía un mensaje a la conversación actual.
- **Requiere Autenticación:** Sí (JWT).
- **Parámetros de Ruta:**
  - `chatId`: Conversación de destino.
- **Cuerpo de la Petición (`application/json`):**
  ```json
  {
    "content": "¡Nos vemos a las 9!"
  }
  ```
- **Lógica asociada:**
  - Guarda el mensaje en base de datos.
  - Actualiza la propiedad `lastMessage` del chat asociado para mejorar la eficiencia del feed principal.
  - Si el destinatario se encuentra activamente en la misma sala de chat de WebSocket (sala `chatId`), añade al usuario al array de lectura `readBy` de forma automática.
  - Realiza un bucle de emisión de eventos de socket `new_message` a todos los participantes conectados en tiempo real.
- **Respuesta de Éxito (`201 Created`):** Devuelve el objeto del mensaje recién creado y poblado con los datos de perfil mínimos del remitente.

---

### 5.6 Planes y Eventos (Plans)

#### `GET /api/plans`
- **Descripción:** Obtiene la lista (Feed) con todas las propuestas de planes sociales del ecosistema de manera cronológica de cara al usuario.
- **Requiere Autenticación:** Sí (JWT).
- **Respuesta de Éxito (`200 OK`):** Devuelve un array de objetos de tipo `Plan` poblando la información mínima de su respectivo creador.

#### `POST /api/plans`
- **Descripción:** Publica una nueva propuesta de plan social al feed colectivo.
- **Requiere Autenticación:** Sí (JWT).
- **Cuerpo de la Petición (`application/json`):**
  ```json
  {
    "title": "Cena en el centro",
    "description": "Vamos a probar el nuevo restaurante italiano que acaban de abrir.",
    "emojiIcon": "🍕",
    "datetime": "2026-06-15T21:00:00Z",
    "location": "Restaurante La Tagliatella, Ciudad Real"
  }
  ```
- **Lógica asociada:**
  1. Registra el `Plan` en la base de datos con el usuario actual marcado como asistente inicial (`attendees = [creator]`).
  2. Genera de forma automática una conversación grupal (`Chat` con propiedad `isGroup = true`, `name = req.body.title` y `emojiIcon = req.body.emojiIcon`).
  3. Añade el identificador de esta conversación (`chatId`) dentro del plan recién creado.
  4. Realiza una búsqueda automática de usuarios con intereses en común que coincidan con la temática del plan y les envía una notificación de invitación (`plan_interest`).
- **Respuesta de Éxito (`201 Created`):** Retorna el documento del plan.

#### `POST /api/plans/:id/join`
- **Descripción:** Permite unirse o desapuntarse de un plan propuesto (acción de tipo alternante / Toggle).
- **Requiere Autenticación:** Sí (JWT).
- **Parámetros de Ruta:**
  - `id`: Identificador del Plan.
- **Lógica asociada (al unirse):**
  - Añade el ID del usuario al array `attendees` del plan.
  - Vincula de forma directa al usuario con el `Chat` de grupo de coordinación de dicho plan (`participants.push(userId)`).
  - Envía notificaciones persistentes (`plan_join`) y lanza el evento real-time `plan_join` vía WebSocket al creador y asistentes anteriores del plan.
- **Lógica asociada (al desapuntarse):**
  - Remueve al usuario de `attendees`.
  - Remueve al usuario de la lista de participantes del `Chat` grupal del plan.
- **Respuesta de Éxito (`200 OK`):** Retorna el plan modificado con el array de asistentes actualizado.

#### `POST /api/plans/:id/react`
- **Descripción:** Permite reaccionar a un plan concreto con un emoji o remover la reacción si ya se había reaccionado anteriormente con el mismo ( Toggle ).
- **Requiere Autenticación:** Sí (JWT).
- **Parámetros de Ruta:**
  - `id`: Identificador del Plan.
- **Cuerpo de la Petición (`application/json`):**
  ```json
  {
    "emoji": "❤️"
  }
  ```
- **Respuesta de Éxito (`200 OK`):** Devuelve el objeto del plan actualizado.

#### `POST /api/plans/:id/comments`
- **Descripción:** Inserta un nuevo comentario de texto en el hilo de conversación y debate público del plan.
- **Requiere Autenticación:** Sí (JWT).
- **Parámetros de Ruta:**
  - `id`: Identificador del Plan.
- **Cuerpo de la Petición (`application/json`):**
  ```json
  {
    "text": "¡Qué buen plan! Me apunto."
  }
  ```
- **Lógica asociada:**
  - Almacena el comentario en el array de comentarios internos del plan.
  - Emite un evento WebSocket en tiempo real `plan_comment` al organizador y los asistentes del evento.
  - Crea una notificación física persistente `plan_comment` para todos los destinatarios implicados que sean diferentes al autor del propio comentario.
- **Respuesta de Éxito (`201 Created`):** Retorna el plan actualizado con el nuevo comentario en el listado.

#### `GET /api/plans/:id/comments`
- **Descripción:** Obtiene los comentarios de un plan ordenados de forma cronológica, poblando el nombre de usuario y su foto de perfil para pintarlos cómodamente.
- **Requiere Autenticación:** Sí (JWT).
- **Parámetros de Ruta:**
  - `id`: Identificador del Plan.
- **Respuesta de Éxito (`200 OK`):** Array de comentarios.

---

### 5.7 Notificaciones (Notifications)

#### `GET /api/notifications`
- **Descripción:** Obtiene los últimos 50 avisos de actividad y notificaciones generados para el usuario en orden inverso de tiempo.
- **Requiere Autenticación:** Sí (JWT).
- **Respuesta de Éxito (`200 OK`):** Array de objetos del modelo `Notification`.

#### `GET /api/notifications/unread-count`
- **Descripción:** Devuelve un conteo numérico rápido de los avisos y notificaciones que permanecen sin leer por el usuario. Útil para mostrar un globo rojo o badge numérico en el cliente.
- **Requiere Autenticación:** Sí (JWT).
- **Respuesta de Éxito (`200 OK`):**
  ```json
  {
    "count": 3
  }
  ```

#### `PUT /api/notifications/read-all`
- **Descripción:** Transforma todos los avisos y notificaciones que tenía pendientes el usuario a estado leído (`read = true`).
- **Requiere Autenticación:** Sí (JWT).
- **Respuesta de Éxito (`200 OK`):**
  ```json
  {
    "message": "Todas las notificaciones han sido marcadas como leídas."
  }
  ```

#### `PUT /api/notifications/:id/read`
- **Descripción:** Marca un aviso de notificación en particular como leído.
- **Requiere Autenticación:** Sí (JWT).
- **Parámetros de Ruta:**
  - `id`: Identificador de la notificación.
- **Respuesta de Éxito (`200 OK`):** Retorna el documento de notificación actualizado.

---

## 6. Comunicación en Tiempo Real (WebSockets / Socket.io)

La infraestructura en tiempo real se encarga de sincronizar el estado, los mensajes del chat y las alertas reactivas sin necesidad de realizar polling de red de forma periódica.

### Conexión y Control de Sala (Salas de Socket)
1. **Sala Privada de Usuario:** Al conectarse un cliente con éxito, el servidor Express le une automáticamente a una sala con el nombre correspondiente a su identificador de usuario (`userId`). Esto permite que el backend pueda redirigir eventos exclusivamente a un usuario usando:
   `io.to(userId.toString()).emit(event, data)`
2. **Salas de Chat:** Cuando el cliente visualiza o entra a un chat en particular, emite un mensaje `join_chat` para ser unido a la sala del chat (`chatId`). Al cerrar la pantalla del chat, emite un `leave_chat` para abandonar la sala.

---

### Tabla resumen de eventos WebSocket

#### Eventos Cliente-a-Servidor (Entrada)

| Evento | Parámetro | Acción del Servidor |
| :--- | :--- | :--- |
| `join_chat` | `chatId` (String) | Une al cliente a la sala del chat `chatId` para recibir mensajes. Paralelamente, actualiza masivamente todos los mensajes pendientes de ese chat para registrarlos como leídos por este usuario (`readBy`). |
| `leave_chat`| `chatId` (String) | Saca al cliente de la sala de socket asociada al chat `chatId`. Deja de recibir mensajes en tiempo real para esa conversación. |

#### Eventos Servidor-a-Cliente (Salida)

##### `user_status_change`
- **Descripción:** Notificación masiva global que informa al resto sobre cambios en el estado de conexión de un usuario.
- **Payload de datos:**
  ```json
  {
    "userId": "60b8d2f5f3b3b211a8c1f9b1",
    "status": "online", // o "offline"
    "connectedAt": "2026-07-01T12:00:00.000Z" // o "disconnectedAt"
  }
  ```

##### `initial_active_users`
- **Descripción:** Enviado individualmente al cliente que se acaba de conectar con éxito. Le brinda la lista completa con los IDs de todos los usuarios que están actualmente en línea.
- **Payload de datos:** Array de strings con los IDs de los usuarios activos:
  ```json
  ["60b8d2f5f3b3b211a8c1f9b1", "60b8d2f5f3b3b211a8c1f9b5"]
  ```

##### `new_message`
- **Descripción:** Mensaje recibido dentro de un chat. Se emite a todos los participantes del chat.
- **Payload de datos:**
  ```json
  {
    "chatId": "60b8d2f5f3b3b211a8c1fb10",
    "message": {
      "_id": "60b8d2f5f3b3b211a8c1fb55",
      "chatId": "60b8d2f5f3b3b211a8c1fb10",
      "sender": {
        "_id": "60b8d2f5f3b3b211a8c1f9b1",
        "username": "juan_perez"
      },
      "content": "¡Hola a todos!",
      "readBy": ["60b8d2f5f3b3b211a8c1f9b1"],
      "createdAt": "2026-07-01T12:12:00.000Z"
    }
  }
  ```

##### `plan_join`
- **Descripción:** Enviado en tiempo real a los interesados de un plan cuando alguien nuevo se une al evento.
- **Payload de datos:**
  ```json
  {
    "planId": "60b8d2f5f3b3b211a8c1fc01",
    "planTitle": "Cena en el centro",
    "userId": "60b8d2f5f3b3b211a8c1f9b5",
    "userName": "sofia_gomez"
  }
  ```

##### `plan_comment`
- **Descripción:** Enviado en tiempo real a los asistentes e interesados de un plan cuando se publica un nuevo comentario en la propuesta.
- **Payload de datos:**
  ```json
  {
    "planId": "60b8d2f5f3b3b211a8c1fc01",
    "planTitle": "Cena en el centro",
    "comment": {
      "user": {
        "_id": "60b8d2f5f3b3b211a8c1f9b5",
        "username": "sofia_gomez",
        "avatar": "avatar_girl_3.png"
      },
      "text": "¡Me encanta! Llevaré postre.",
      "createdAt": "2026-07-01T12:15:00.000Z"
    }
  }
  ```

##### `new_notification`
- **Descripción:** Notificación reactiva persistente en tiempo real. Se le envía a la sala privada del usuario receptor.
- **Payload de datos:** Objeto de tipo `Notification` completo que acaba de guardarse en la base de datos de MongoDB.
  ```json
  {
    "_id": "60b8d2f5f3b3b211a8c1fa80",
    "recipient": "60b8d2f5f3b3b211a8c1f9b1",
    "sender": "60b8d2f5f3b3b211a8c1f9b5",
    "type": "plan_join",
    "message": "sofia_gomez se ha unido al plan: Cena en el centro",
    "referenceId": "60b8d2f5f3b3b211a8c1fc01",
    "read": false,
    "createdAt": "2026-07-01T12:12:00.000Z"
  }
  ```
