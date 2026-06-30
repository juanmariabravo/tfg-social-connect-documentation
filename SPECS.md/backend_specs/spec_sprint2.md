# Especificación Técnica: Sprint 2 - Usuarios y Perfiles
**Fechas:** 04/05/2026 - 18/05/2026
**Objetivo:** Implementar el sistema de usuarios con autenticación JWT y gestión de perfiles.

---

## 1. Alcance del Sprint
El objetivo principal es implementar el sistema de registro/login y la gestión de perfiles de usuario.

## 2. Requisitos Técnicos

### 2.1 Esquema de Usuario (MongoDB)
- **Colección:** `users`
- **Campos:**
  - `username` (String, required)
  - `email` (String, required, unique)
  - `password` (String, required, encriptada con bcrypt)
  - `dateOfBirth` (String, required) - fecha de nacimiento del usuario (ISO string)
  - `createdAt` (Date)
  - `updatedAt` (Date)

**Notas:** 
* Los intereses se movieron al modelo Profile. El modelo Interest fue eliminado.
* El campo `name` fue renombrado a `username` para unificar la identidad del usuario.

### 2.2 Esquema de User Profile (MongoDB)
- **Colección:** `profiles`
- **Campos:**
  - `userId` (ObjectId, ref: User, unique)
  - `bio` (String)
  - `location` (String) - ubicación del usuario
  - `avatar` (String, URL) - foto virtual
  - `photo` (String, URL) - foto real del usuario
  - `interests` (Array of Objects) - temas de interés del usuario
    - `name` (String)
    - `emoji` (String)
  - `personality` (Object): atributos de personalidad (Big Five)
    - `extroversion` (Number, 1-10)
    - `openness` (Number, 1-10)
    - `conscientiousness` (Number, 1-10)
    - `agreeableness` (Number, 1-10)
    - `neuroticism` (Number, 1-10)
  - `onboardingCompleted` (Boolean, default: false) - para saber si completó la encuesta
  - `createdAt` (Date)
  - `updatedAt` (Date)

## 3. Endpoints a implementar

### 3.1 Autenticación

#### POST /api/auth/register
- **Descripción:** Registrar un nuevo usuario.
- **Cuerpo de la petición:**
  ```json
  {
    "username": "string",
    "email": "string",
    "password": "string",
    "dateOfBirth": "YYYY-MM-DD"
  }
  ```
- **Respuesta:** Token JWT + datos del usuario.
- **Nota:** Al registrarse se crea automáticamente un perfil vacío vinculado al usuario.

#### POST /api/auth/login
- **Descripción:** Iniciar sesión.
- **Cuerpo de la petición:**
  ```json
  {
    "email": "string",
    "password": "string"
  }
  ```
- **Respuesta:** Token JWT + datos del usuario.

### 3.2 Gestión de Perfil

#### GET /api/profiles/:userId
- **Descripción:** Obtener el perfil de un usuario. Incluye datos del usuario poblados.
- **Autenticación:** Required (JWT).
- **Respuesta:** Datos del perfil + objeto `user`.

#### PUT /api/profiles/:userId
- **Descripción:** Actualizar el perfil del usuario autenticado. También actualiza el `username` en el modelo User.
- **Autenticación:** Required (JWT).
- **Cuerpo de la petición:**
  ```json
  {
    "username": "string",
    "bio": "string",
    "location": "string",
    "avatar": "string",
    "photo": "string",
    "interests": [
      { "name": "string", "emoji": "string" }
    ],
    "personality": {
      "extroversion": 1-10,
      "openness": 1-10,
      "conscientiousness": 1-10,
      "agreeableness": 1-10,
      "neuroticism": 1-10
    }
  }
  ```

### 3.3 Documentación de API (Swagger UI)
- **URL:** `/docs`
- **Herramientas:** `swagger-jsdoc`, `swagger-ui-express`.
- **Funcionalidad:** 
  - Visualización interactiva de todos los endpoints.
  - Soporte para autenticación JWT (Authorize con Bearer token).
  - Esquemas de modelos y validaciones Joi integrados en la documentación.

## 4. Middleware y Robustez

### 4.1 authMiddleware
- **Descripción:** Verificar el token JWT en las cabeceras.
- **Ubicación:** `/middleware/auth.js`
- **Lógica:**
  1. Extraer token de la cabecera `Authorization: Bearer <token>`.
  2. Verificar token con la clave secreta.
  3. Adjuntar datos del usuario a `req.user`.
  4. Llamar a `next()` o devolver error 401.

### 4.2 Validación de Parámetros (Robustez)
- **Lógica:** Validación manual de `ObjectIds` en parámetros de ruta (ej. `:userId`).
- **Objetivo:** Evitar errores 500 generados por `CastError` de Mongoose cuando se proporcionan IDs mal formados.
- **Resultado:** Retornar 404 (Not Found) de forma controlada.

## 5. Estrategia de Pruebas (TDD)

### Test 1: Registro de usuario
- **Framework:** Jest
- **Workflow:**
  1. Definir test para `/api/auth/register` (Fase Red).
  2. Verificar que el usuario se guarda con contraseña encriptada (bcrypt).
  3. Implementar ruta y controlador (Fase Verde).
  4. Refactorizar código si es necesario.

### Test 2: Acceso protegido
- **Workflow:**
  1. Definir test para `/api/profiles/:userId` sin token (Fase Red).
  2. Verificar que devuelve error 401.
  3. Implementar middleware de auth (Fase Verde).
  4. Refactorizar código si es necesario.

## 6. Librerías requeridas
- `bcryptjs` - Encriptación de contraseñas.
- `jsonwebtoken` - Generación y verificación de JWT.
- `joi` - Validación de datos de entrada.
- `mongodb-memory-server` - Base de datos en memoria para tests (dev dependencies).

## 6.1 Testing
- Los tests utilizan `mongodb-memory-server` para crear una base de datos efímera en memoria.
- Esto evita modificar la base de datos local durante la ejecución de tests.
- Configuración en `jest.config.js` y `setUpTest.js`.

## 7. Definición de Hecho (Definition of Done)
- [x] El esquema de User Profile está implementado.
- [x] El sistema de registro/login funciona con JWT.
- [x] El middleware de auth protege las rutas necesarias.
- [x] El CRUD de perfil (crear, leer, editar) está operativo.
- [x] Los tests de registro y acceso pasan en verde (26 tests).
- [x] Los tests usan mongodb-memory-server (no modifican BD local).
- [x] Documentación interactiva con Swagger UI disponible en `/docs`.
- [x] Validación de robustez para IDs en controladores implementada.
- [x] El código está subido a una rama `feature/` y mergeado a `main` tras validar.
- [x] Este documento `spec_sprint2.md` ha sido actualizado con los cambios realizados.

*La documentación refleja siempre la realidad del código.*