---
name: version-control
description: "Esta skill encapsula el flujo de control de versiones avanzado con Git y GitHub, incluyendo Git Flow/GitHub Flow, Git Submodules, commits semánticos y automatización de despliegue."
risk: safe
source: "NebriAmazon DevOps Core"
date_added: "2026-05-20"
---

# 📦 Skill de Control de Versiones: Git & GitHub Flow

Esta skill establece las directrices de nivel senior para gestionar el ciclo de vida del código fuente a través del sistema de control de versiones **Git** y la plataforma **GitHub**. Define las reglas de ramificación, mensajería semántica, revisión de Pull Requests y el control específico de submódulos en repositorios distribuidos.

---

## 🧠 Filosofía de Integración Continua

> "Un historial de Git limpio y bien estructurado no es solo un registro del pasado, sino la hoja de ruta y la garantía de estabilidad y colaboración para el futuro del proyecto."

---

## 🌿 1. Estrategia de Ramificación (GitHub Flow)

Para mantener una entrega continua ágil y sin bloqueos, adoptamos **GitHub Flow**:

*   **Rama Principal (`main`):** El código en `main` siempre debe ser desplegable y estar en un estado verde de tests al 100%.
*   **Ramas de Características (`feature/`):**
    *   Creadas a partir de `main` para implementar una característica o corrección de error (ej. `feature/auth-jwt`, `bugfix/cart-item-removal`).
    *   Tienen nombres descriptivos, breves y en minúsculas.
*   **Ramas de Pruebas o QA (opcional):** Para consolidar cambios antes del merge a `main`.

---

## 📝 2. Convenciones de Commit Semántico (Conventional Commits)

Cada commit debe ser atómico y describir con precisión el cambio introducido, siguiendo la convención de **Conventional Commits**:

```text
<tipo>(<alcance>): <descripción corta en minúsculas>

[cuerpo detallado opcional explicando el porqué del cambio]
```

### Tipos Comunes:
*   `feat`: Una nueva característica para el usuario.
*   `fix`: Una corrección de un bug.
*   `docs`: Cambios únicamente en la documentación.
*   `style`: Cambios cosméticos que no alteran la lógica del código (espaciados, formateo, punto y coma).
*   `refactor`: Refactorización de código que no añade características ni corrige bugs.
*   `test`: Añadir o corregir pruebas unitarias o de integración.
*   `chore`: Tareas de mantenimiento, actualización de dependencias o configuraciones del build.

---

## 🤝 3. Gestión y Revisión de Pull Requests (PR)

Las Pull Requests en GitHub actúan como la última línea de defensa técnica:

*   **Descripción Clara:** Cada PR debe resumir qué cambios introduce, qué PRD resuelve y qué pruebas se han ejecutado.
*   **Aprobaciones Rigurosas:** Requiere la validación del agente de calidad antes de autorizar el merge.
*   **Merge Limpio (Squash and Merge):** Preferible para fusionar ramas de características hacia `main` con el fin de mantener un historial lineal y limpio.

---

## 🔗 4. Control Avanzado de Submódulos de Git

En arquitecturas modulares (como NebriAmazon, que separa frontend y backend en submódulos), el control exacto es vital:

*   **Actualización de Punteros:** Cada vez que se actualiza o commitea código dentro de un submódulo (`frontend-nebri-amazon` o `backend-nebri-amazon`), se debe actualizar la referencia en el repositorio principal orquestador.
*   **Clonación Recursiva:** El código principal debe clonarse con `--recursive` para descargar los submódulos.
*   **Comandos Críticos:**
    *   Sincronizar cambios: `git submodule update --remote --merge`
    *   Estado de submódulos: `git submodule status`

---

## 🛠️ Checklist del Gestor de Versiones

- [ ] ¿He creado la rama de característica a partir de la versión más reciente de `main`?
- [ ] ¿Los commits se han estructurado siguiendo el estándar de Commits Semánticos?
- [ ] ¿Cada commit es atómico (realiza un único cambio conceptual limpio)?
- [ ] ¿Se han actualizado correctamente las referencias de los submódulos de Git en el repositorio raíz?
- [ ] ¿La Pull Request en GitHub contiene la descripción técnica detallada y está enlazada al PRD correspondiente?
- [ ] ¿Se han resuelto todos los conflictos de mezcla de forma segura y se ha verificado que los tests sigan en verde?
