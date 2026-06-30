# Especificación Técnica: Sprint 5 - Frontend: Exploración y Perfil Público
**Fechas:** 15/06/2026 - 28/06/2026
**Estado:** En Desarrollo ⌚

---

## 0. Fase de Diseño (Mockups)
**Herramienta:** Figma / Lovable
**Objetivo:** Diseñar la experiencia de descubrimiento y la visualización de otros perfiles.

### 0.1 Mockups a Implementar
- [ ] **Explorar:** Vista de búsqueda con filtros rápidos (Cerca, Intereses, Afinidad).
- [ ] **Perfil Público:** Vista detallada de otro usuario (Bio, Intereses, Gráfico de Personalidad) con botón de acción (Conectar/Mensaje).

---

## 1. Alcance del Sprint
Transformar la aplicación en una red social completa permitiendo a los usuarios buscar y descubrir nuevas personas basándose en criterios reales de compatibilidad.

## 2. Requisitos Técnicos
- **Visualización de Datos:** Gráficos de radar o barras para la personalidad (PersonalityChart).
- **Filtrado Dinámico:** Gestión de estado para los filtros de exploración.
- **Navegación:** Rutas dinámicas para perfiles externos `/u/:id`.

## 3. Vistas y Componentes Clave

### 3.1 Explorar (`src/pages/Explore.tsx`)
- **Funcionalidad:**
  - Consumir el endpoint `GET /api/explore`.
  - Selector de filtros (Pestañas o Chips): "Para ti", "Cerca", "Intereses", "Personalidad", "Edad".
  - Barra de búsqueda por nombre/username.
  - Visualización en Grid o Lista de perfiles con el % de afinidad visible.

### 3.2 Perfil de Usuario (`src/pages/UserProfile.tsx`)
... (se mantiene igual para perfiles públicos)

### 3.3 Mi Perfil y Onboarding (Mejoras)
- **Geolocalización por Navegador:** En la edición de perfil y onboarding, añadir un botón para obtener la ubicación actual mediante la API de Geolocation del navegador. Si el usuario da permiso, el campo `location` se rellenará automáticamente con el nombre de la ciudad/zona detectada (vía geocodificación inversa si es necesario).

## 4. Definición de Hecho (Definition of Done)
- [ ] Página de Explorar operativa con filtros y búsqueda.
- [ ] Visualización correcta del score de afinidad.
- [ ] Página de Perfil Público completa con gráficos de personalidad.
- [ ] Flujo de "Conectar" desde Explorar verificado.
- [ ] UI consistente con el resto de la aplicación.
- [ ] Entrega final y bloqueo de código.
