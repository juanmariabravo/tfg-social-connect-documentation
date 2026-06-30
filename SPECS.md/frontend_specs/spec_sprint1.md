# Especificación Técnica: Sprint 1 - Frontend Base
**Fechas:** 20/04/2026 - 03/05/2026
**Objetivo:** Configuración del entorno React y validación de conexión con la API REST.

---

## 1. Alcance del Sprint
Preparar el entorno de desarrollo del cliente, configurar la estructura de componentes y verificar que el frontend puede "hablar" con el backend.

## 2. Requisitos Técnicos
- **Framework:** React.js.
- **Estilos:** Tailwind CSS o CSS Modules.
- **Cliente HTTP:** Axios.
- **Enrutamiento:** React Router.

## 3. Estructura de Componentes propuesta
- `/src/components`: Componentes reutilizables.
- `/src/pages`: Vistas principales (Login, Registro, Home).
- `/src/hooks`: Lógica de estado compartida.
- `/src/services`: Configuración de llamadas a la API (Axios instance).

## 4. Funcionalidades a implementar
- **Setup de Proyecto:** Inicialización de React (Vite/Create React App).
- **Conectividad:** Configurar `axios` con la URL base del backend (`http://localhost:5000/api`).
- **Vista de Prueba:** Crear un componente simple que consuma el endpoint `GET /api/status` del backend y muestre el estado en pantalla.

## 5. Estrategia de Pruebas (Visual/Funcional)
- **Workflow:**     
    1. Implementar servicio de conexión con el backend.
    2. Crear componente `StatusChecker`.
    3. Verificar que el componente muestra "Online" al recibir respuesta 200.

## 6. Definición de Hecho (Definition of Done)
- [x] Entorno React configurado y compilando correctamente.
- [x] El componente de prueba recibe el JSON del backend correctamente.
- [x] El código está subido a la rama `feature/` y mergeado a `main`.

## Estado del Sprint
Todas las tareas han sido completadas con éxito a fecha de cierre del sprint (04/05/2026). El frontend está listo para comenzar a desarrollar las funcionalidades específicas de la aplicación en el próximo sprint.