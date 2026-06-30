# Especificación Técnica: Sprint 1 - Infraestructura Base
**Fechas:** 20/04/2026 - 03/05/2026
**Objetivo:** Establecer los cimientos del backend (MERN), la arquitectura MVC y la conectividad con la base de datos.

---

## 1. Alcance del Sprint
El objetivo principal es preparar el entorno de desarrollo y verificar la comunicación cliente-servidor-base de datos.

## 2. Arquitectura del Sistema
Se implementará el patrón **Modelo-Vista-Controlador (MVC)** para garantizar una separación de responsabilidades clara y facilitar el mantenimiento del TFG.



[Image of MVC architecture diagram]


### Estructura de carpetas requerida:
- `/controllers`: Lógica de negocio (procesamiento de peticiones).
- `/models`: Definición de esquemas de datos (MongoDB/Mongoose).
- `/routes`: Definición de endpoints de la API.
- `/config`: Configuración de conexión (DB, variables de entorno).
- `/middleware`: Funciones intermedias (auth, validación).
- `/tests`: Archivos de pruebas unitarias/integración (TDD).

## 3. Requisitos Técnicos
- **Entorno:** Node.js + Express.js.
- **Base de Datos:** MongoDB.
- **Gestión de variables:** Uso de `.env` (debe estar en `.gitignore`).
- **Control de versiones:** Git con ramas para cada tarea.

## 4. Endpoints a implementar (TDD)
En este sprint, el único endpoint obligatorio es el de "Health Check".

### GET /api/status
- **Descripción:** Verifica que el servidor está operativo y conectado a la BD.
- **Respuesta esperada:**
  ```json
  {
    "status": "online",
    "timestamp": "YYYY-MM-DDTHH:MM:SSZ",
    "database": "connected"
  }
  ```

## 5. Estrategia de Pruebas (TDD)
- **Framework:** Jest
- **Workflow:** 1. Definir test para `/api/status` (Fase Red).
  1. Implementar ruta y controlador mínimos (Fase Verde).
  2. Refactorizar código si es necesario.

## 6. Definición de Hecho (Definition of Done)
- [x] La estructura de carpetas (MVC) está creada y operativa.
- [x] La conexión a MongoDB se realiza sin errores.
- [x] El test para `GET /api/status` se ejecuta y pasa en verde.
- [x] El código está subido a una rama `feature/` y mergeado a `main` tras validar.
- [x] Este documento `spec_sprint1.md` ha sido actualizado si hubo cambios en el alcance.

*Si durante el desarrollo ves que necesitas una carpeta nueva o un middleware extra, actualiza este archivo `.md`. ("Spec-driven development": **la documentación refleja siempre la realidad del código**.)*

## Estado del Sprint
Todas las tareas han sido completadas con éxito a fecha de cierre del sprint (04/05/2026). El backend está listo para comenzar a implementar funcionalidades específicas en el próximo sprint.
