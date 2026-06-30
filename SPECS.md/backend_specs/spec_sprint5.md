# Especificación Técnica: Sprint 5 - Backend: Algoritmo de Afinidad y Exploración
**Fechas:** 15/06/2026 - 28/06/2026
**Estado:** En Desarrollo ⌚

---

## 1. Alcance del Sprint
El objetivo final del backend es implementar el motor de matching "Weighted Compatibility Score" (WCScore) que conecta a los usuarios por afinidad real, basándose en su personalidad, intereses y ubicación. Se proveerá un endpoint de exploración avanzado con capacidades de filtrado y ordenación dinámica.

## 2. Requisitos Técnicos
- **Lógica de Negocio:** Implementación de funciones de cálculo de distancia y similitud.
- **Consultas Avanzadas:** Aggregation Framework de MongoDB para filtrado y $sample.
- **Geolocalización:** Uso de coordenadas (si están disponibles) o distancias calculadas entre ciudades/puntos.

## 3. Algoritmo: Weighted Compatibility Score (WCScore)

El score se calcula sobre una base de 100 puntos utilizando la siguiente ponderación:
- **Personalidad (35%):** Basado en el modelo Big Five.
- **Intereses (20%):** Basado en la cantidad de intereses comunes.
- **Cercanía (30%):** Basado en la proximidad geográfica.
- **Edad (15%):** Basado en la similitud de edad (menor diferencia de edad, mayor puntuación).

### 3.1 Funciones Auxiliares
- `getCompatibilityByPersonality(userA, userB)`: Calcula la distancia euclidiana entre los vectores de personalidad.
- `getCompatibilityByInterests(userA, userB)`: Calcula el índice de Jaccard o similar para los arrays de intereses.
- `getCompatibilityByProximity(userA, userB)`: Calcula el score basado en la distancia física.
- `getCompatibilityByAge(userA, userB)`: Calcula el score basado en la diferencia de años entre usuarios.

## 4. Endpoints a Implementar

### 4.1 Exploración (`/api/explore`)

#### GET /api/explore
- **Descripción:** Obtener lista de usuarios recomendados.
- **Parámetros (Query):**
  - `filter`: ['all', 'proximity', 'interests', 'personality', 'age'] (Por defecto: 'all' -> usa WCScore).
  - `search`: String (Búsqueda por nombre o intereses).
  - `limit`: Number (Paginación).
- **Lógica:**
  1. Filtrar usuarios que ya son amigos o tienen solicitudes pendientes.
  2. Aplicar la estrategia de ordenación según el parámetro `filter`. Si el filtro es `age`, se prioriza la compatibilidad por edad.
  3. Devolver perfiles enriquecidos con el porcentaje de afinidad calculado (WCScore).

## 5. Estrategia de Pruebas (TDD)
- **Cálculos Matemáticos:** `tests/matching.test.js` (Validar que el WCScore es coherente).
- **Endpoint de Exploración:** `tests/explore.test.js` (Validar filtros y exclusión de amigos).

## 6. Definición de Hecho (Definition of Done)
- [ ] Lógica de WCScore implementada y testeada.
- [ ] Endpoint `/api/explore` funcional con todos los filtros.
- [ ] Exclusión correcta de usuarios ya vinculados (amigos/pendientes).
- [ ] Documentación de la API actualizada.
