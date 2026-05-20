---
name: backend-development
description: 'Esta skill encapsula las prácticas y patrones de ingeniería senior para el desarrollo backend utilizando Ruby y PostgreSQL, enfocándose en la modularidad, escalabilidad y optimización de bases de datos.'
risk: safe
source: 'NebriAmazon Senior Developer Core'
date_added: '2026-05-20'
---

# 💎 Skill de Desarrollo Backend: Ruby & PostgreSQL

Esta skill define las directrices de arquitectura, diseño de bases de datos y buenas prácticas de desarrollo para construir sistemas backend altamente escalables, seguros y mantenibles utilizando el lenguaje **Ruby** y el gestor de bases de datos **PostgreSQL**.

---

## 🧠 Filosofía Core del Backend Senior

> "El buen código backend no solo debe funcionar de manera correcta, sino que debe ser legible, escalable ante cargas masivas y eficiente con el uso de recursos y accesos a bases de datos."

---

## 1. 🔴 Programación Profesional en Ruby

El código escrito en Ruby debe aprovechar la expresividad y elegancia del lenguaje sin sacrificar el rendimiento:

*   **Orientación a Objetos Pura (OOP):** Clases pequeñas con una única responsabilidad (SRP). Abstracción de lógica compleja en *Services*, *Interactors* o *Query Objects* en lugar de saturar los modelos o controladores.
*   **Convenciones de Estilo (Ruby Style Guide):**
    *   Uso consistente de `snake_case` para métodos y variables, y `CamelCase` para clases y módulos.
    *   Uso de bloques implícitos y métodos funcionales (`map`, `reduce`, `select`, `compact`) para mayor claridad.
*   **Manejo Elegante de Excepciones:**
    *   Evitar rescatar la clase genérica `Exception`. Capturar siempre excepciones específicas (`StandardError`, `ActiveRecord::RecordNotFound`, `PG::Error`).
    *   Utilizar bloques `begin-rescue-ensure` con responsabilidad, asegurando el cierre de sockets y descriptores de archivos.
*   **Inmutabilidad y Seguridad:** Congelar cadenas de texto constantes (`.freeze`) y evitar mutar argumentos pasados a métodos.

---

## 2. 🐘 Diseño y Optimización de PostgreSQL

Como programador senior, el diseño de la base de datos es la base de todo el sistema:

*   **Diseño de Esquemas Eficiente:**
    *   Elección correcta de tipos de datos (`uuid` para identificadores primarios distribuidos, `jsonb` para datos semiestructurados con capacidad de indexación, `timestamptz` para marcas temporales con huso horario).
    *   Normalización inteligente (generalmente 3NF) y desnormalización controlada mediante caché si es crítico para el rendimiento.
*   **Estrategia de Indexación:**
    *   Indexar siempre claves foráneas (`FOREIGN KEY`) para acelerar los *JOIN*.
    *   Uso de índices parciales (`WHERE active = true`) para reducir el tamaño del índice.
    *   Índices compuestos y de cobertura cuidando el orden de las columnas según las consultas más frecuentes.
*   **Optimización de Consultas:**
    *   Uso crítico de `EXPLAIN ANALYZE` para diagnosticar cuellos de botella y escaneos de tabla secuenciales (*sequential scans*).
    *   Evitar a toda costa el problema del **N+1** mediante carga ansiosa (*eager loading* con `includes`, `preload` o `eager_load`).
*   **Migraciones de Base de Datos Seguras:**
    *   Migraciones no bloqueantes en producción (añadir columnas con valores por defecto `DEFAULT NULL` y aplicar restricciones después).
    *   Uso de transacciones en migraciones para evitar estados inconsistentes ante fallos.

---

## 3. 🧪 Pruebas Automatizadas y Calidad de Código

Un backend profesional no está completo sin una suite de pruebas robusta:

*   **RSpec para BDD (Behavior-Driven Development):**
    *   Pruebas unitarias completas para servicios e interactores.
    *   Pruebas de integración o *request specs* para validar el comportamiento real de los endpoints API y sus respuestas HTTP.
*   **Pruebas de Base de Datos Limpias:**
    *   Uso de transacciones en cada caso de prueba (`DatabaseCleaner` o transacciones transitorias de RSpec) para garantizar que la base de datos comience limpia en cada test.
*   **Mocks y Stubs Controlados:**
    *   Simular peticiones externas a APIs mediante herramientas como `WebMock` o `VCR` para mantener los tests rápidos y deterministas.

---

## 4. 🔒 Seguridad y Rendimiento

*   **Prevención de Vulnerabilidades:**
    *   **Inyección SQL:** No concatenar variables directamente en cadenas SQL. Usar siempre marcadores de parámetros o marcadores de posición provistos por ActiveRecord / Sequel.
    *   **Autenticación y Autorización:** Implementar JWT (JSON Web Tokens) firmados con algoritmos seguros o sesiones encriptadas. Implementar control de acceso basado en roles (RBAC).
*   **Gestión del Pool de Conexiones:**
    *   Ajustar adecuadamente el tamaño del *connection pool* del backend en consonancia con la capacidad máxima de conexiones configurada en PostgreSQL.

---

## 🛠️ Checklist del Programador Senior

- [ ] ¿Las consultas a la base de datos están libres del problema de consultas N+1?
- [ ] ¿He verificado el rendimiento de las consultas críticas usando `EXPLAIN ANALYZE`?
- [ ] ¿Las claves foráneas y columnas de búsqueda frecuente cuentan con índices adecuados?
- [ ] ¿He evitado concatenar variables directamente en strings SQL (prevención de Inyección SQL)?
- [ ] ¿Se capturan y manejan excepciones específicas de Ruby y PostgreSQL de forma controlada?
- [ ] ¿La lógica de negocio pesada reside en clases de servicio y no en los modelos o controladores?
- [ ] ¿El código cuenta con cobertura de pruebas unitarias y de integración en RSpec?
