El @spec# Especificación Técnica: Sprint 2 - Frontend: Usuarios y Perfiles
**Fechas:** 04/05/2026 - 18/05/2026
**Objetivo:** Implementar la interfaz de usuario para autenticación y gestión de perfiles utilizando TypeScript y Tailwind CSS 4.

---

## 0. Fase de Diseño (Mockups)
**Herramienta:** Figma / Lovable
**Objetivo:** Crear wireframes y mockups antes de implementar

### 0.1 Mockups creados
- [x] Login (/login) - Wireframe mobile + desktop
- [x] Registro (/register) - Wireframe mobile + desktop
- [x] Perfil (/profile) - Wireframe mobile + desktop
- [x] Editar Perfil (Integrado en /profile) - Wireframe mobile + desktop
- [x] Onboarding (/onboarding) - Wireframe de los pasos 1-3

### 0.2 Entregables
- [x] Links a prototipos en Figma/Lovable
- [x] Capturas para documentación (Carpeta `/Mockups` en la raíz)
- [x] Checklist de componentes por vista

---

## 1. Alcance del Sprint
El objetivo principal es implementar las vistas de login, registro y perfil de usuario, conectándolas con los endpoints del backend del Sprint 2.

## 2. Requisitos Técnicos
- **Framework:** React.js 19 (TypeScript)
- **Gestión de estado:** Context API (`AuthContext.tsx`)
- **Cliente HTTP:** Axios
- **Enrutamiento:** React Router 7
- **Estilos:** Tailwind CSS 4 + Radix UI + Lucide React
- **Gráficos:** Recharts (para el gráfico de personalidad)
- **Autenticación:** Almacenar JWT en `localStorage` (`accessToken`)

## 3. Vistas a implementar

### 3.1 Login (/)
- **Ruta:** `/`
- **Campos:** email, contraseña
- **Funcionalidad:**
  - Enviar credenciales a POST /api/auth/login
  - Almacenar JWT en localStorage
  - Redireccionar a `/onboarding` o `/profile` tras login exitoso
- **Estados:** idle, loading, error

### 3.2 Registro (/register)
- **Ruta:** `/register`
- **Campos:** username, email, contraseña, dateOfBirth
- **Funcionalidad:**
  - Enviar datos a POST /api/auth/register
  - Almacenar JWT en localStorage
  - Redireccionar a `/onboarding` tras registro exitoso
- **Estados:** idle, loading, error

### 3.3 Perfil (/profile)
- **Ruta:** `/profile`
- **Funcionalidad:**
  - Obtener datos del perfil con GET /api/profiles/:userId
  - **Sistema de Fotos Doble:** Alternancia entre Foto Real y Foto Virtual (Avatar).
    - **Lógica de fotos por defecto:** Si no hay foto real, se usa un placeholder de Wikimedia. Si no hay avatar virtual, se usa Gravatar basado en el email.
  - **Edición Integrada:** Botón para activar modo edición en la misma página.
  - Mostrar datos: username, bio, location, intereses (con emojis).
  - Mostrar atributos de personalidad (Big Five) mediante un gráfico de barras horizontales (`PersonalityChart`).
  - Gestión de fotos mediante `PhotoUploadModal`.
- **Estados:** loading, loaded, error

### 3.4 Encuesta de Personalidad e Intereses (/onboarding)
- **Ruta:** `/onboarding`
- **Descripción:** Encuesta dinámica inicial para configurar el perfil tras el primer registro.
- **Trigger:** Se muestra automáticamente si `onboardingCompleted` es `false`.
- **Flujo:**
  1. Configuración del perfil: username, ubicación, descripción y fotos.
  2. Elegir intereses (tags predefinidos + personalizados). Como mínimo 3 intereses.
  3. Test de personalidad (Big Five traits).
- **Funcionalidad:**
  - Guardar en PUT /api/profiles/:userId tras completarse.
  - Marcar perfil como "configurado" (`onboardingCompleted: true`).
- **Estados:** idle, loading, saving, success, error

## 4. Estructura de componentes y archivos
- `/src/pages/Login.tsx`
- `/src/pages/Register.tsx`
- `/src/pages/Profile.tsx` (Incluye lógica de edición)
- `/src/pages/Onboarding.tsx`
- `/src/components/PersonalityChart.tsx` (Bar chart con Recharts)
- `/src/components/PhotoUploadModal.tsx` (Gestión de imágenes)
- `/src/components/AuthBranding.tsx` (Componente visual para Login/Register)
- `/src/components/ui/` (Componentes base: Button, Input, Textarea, Avatar, etc.)
- `/src/context/AuthContext.jsx` (Gestión de estado de autenticación)
- `/src/services/api.js` (Configuración de Axios con interceptor)

## 5. Servicios API
- `/src/services/api.js`: Configuración de Axios con interceptor para incluir JWT en la cabecera `Authorization`.

## 6. Estrategia de Pruebas (Visual/Funcional)
- **Workflow:**
  1. Verificar que el formulario de login envía datos correctamente.
  2. Verificar que el login redirige al onboarding/perfil tras éxito.
  3. Verificar que el perfil muestra correctamente el gráfico de personalidad y las fotos.
  4. Verificar que la edición de perfil (nombre, bio, intereses) persiste en el backend.

## 7. Definición de Hecho (Definition of Done)
### Fase Diseño
- [x] Los mockups de todas las vistas están creados y guardados en `/Mockups`.
- [x] Se han validado los wireframes antes de implementar.

### Fase Implementación
- [x] La vista de Login está operativa y conecta con el backend.
- [x] La vista de Registro está operativa y conecta con el backend.
- [x] La vista de Perfil muestra los datos, el gráfico de personalidad y las fotos.
- [x] La edición de perfil integrada en la página de perfil funciona.
- [x] El Onboarding está completo y marca el perfil como finalizado.
- [x] El token JWT se gestiona correctamente en localStorage.
- [x] El código está migrado a TypeScript (.tsx) casi en su totalidad.
- [x] El código está mergeado a la rama `main`.

*La documentación refleja siempre la realidad del código.*