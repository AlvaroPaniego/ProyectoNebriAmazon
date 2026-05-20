---
name: frontend-development
description: 'Esta skill encapsula las directrices y estándares de ingeniería senior para el desarrollo frontend, enfocándose en Vue 3 (Composition API), gestión de estado con Pinia, compilación rápida con Vite y dominio avanzado de HTML, CSS y JavaScript moderno.'
risk: safe
source: 'NebriAmazon Senior Frontend Core'
date_added: '2026-05-20'
---

# 💎 Skill de Desarrollo Frontend: Vue 3, Pinia & Vite

Esta skill establece las directrices de arquitectura, diseño de interfaces, gestión de estado y rendimiento para construir aplicaciones web del lado del cliente altamente interactivas, rápidas, accesibles y mantenibles.

---

## 🧠 Filosofía del Frontend de Alto Nivel

> "La interfaz de usuario es la cara visible de nuestro producto. Un frontend excelente debe ser estéticamente impecable, reaccionar de manera instantánea, gestionar el estado de forma predecible y ser inclusivo y accesible para todos los usuarios."

---

## 🟢 1. Vue 3 & Composition API (Script Setup)

El desarrollo en Vue 3 debe priorizar la modularidad, reactividad eficiente y legibilidad del código:

*   **Composition API con `<script setup>`:** Es el estándar moderno para escribir componentes legibles, concisos y con mejor rendimiento de compilación.
*   **Composables Reutilizables (`use...`):** Abstraer la lógica reactiva (peticiones API, control de formularios, geolocalización, etc.) en funciones composables independientes y testeables.
*   **Reactividad Optimizada:**
    *   Entender la diferencia y elegir adecuadamente entre `ref` (para valores primitivos o reasignaciones completas) y `reactive` (para objetos de estado agrupado).
    *   Uso cuidadoso de `computed` para evitar cálculos pesados en cada renderizado y cachear valores reactivos.
*   **Ciclo de Vida Limpio:** Liberar recursos (como event listeners, timers, sockets) en `onUnmounted` para prevenir fugas de memoria.

---

## 🍍 2. Gestión de Estado con Pinia

Pinia proporciona un flujo de datos centralizado, predecible y modular:

*   **Tiendas Modulares (Store-per-feature):** Evitar una única tienda monolítica. Crear tiendas independientes para dominios específicos (ej. `useAuthStore`, `useCartStore`, `useProductStore`).
*   **Estructura de la Tienda:**
    *   `state`: El estado mínimo representable (ej. `items: []`).
    *   `getters`: Equivale a propiedades computadas del estado (ej. `totalAmount`).
    *   `actions`: Lógica de negocio y peticiones asíncronas para modificar el estado (ej. `fetchProducts()`).
*   **Mutaciones Controladas:** Modificar el estado siempre a través de acciones cuando involucre lógica compleja o llamadas a APIs para mantener la trazabilidad.

---

## ⚡ 3. Construcción y Optimización con Vite

Vite proporciona un entorno de desarrollo instantáneo y empaquetado optimizado:

*   **Configuración Eficiente (`vite.config.js`):** Uso de alias de rutas (ej. `@` apuntando a `/src`) para evitar rutas relativas complejas (`../../`).
*   **División de Código (Code Splitting):** Implementar carga diferida (*lazy loading*) de rutas utilizando importaciones dinámicas en Vue Router para reducir el tamaño del bundle inicial.
*   **Variables de Entorno:** Consumir variables mediante `import.meta.env.VITE_...` asegurando que no se expongan credenciales críticas del lado del cliente.

---

## 🎨 4. Fundamentos Avanzados (HTML5, CSS3, ES6+)

Un desarrollador senior destaca en las bases nativas de la web:

*   **HTML Semántico & Accesibilidad (a11y):**
    *   Uso de elementos semánticos de HTML5 (`<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<footer>`) para SEO y lectores de pantalla.
    *   Atributos ARIA cuando se creen componentes interactivos personalizados.
*   **Estilos CSS Modernos & Variables:**
    *   CSS Vanilla de alta calidad usando Flexbox y CSS Grid para layouts complejos y adaptables.
    *   Uso de Variables CSS (Custom Properties) para sistemas de diseño coherentes y soporte nativo de modo oscuro/claro de forma fluida.
*   **JavaScript (ES6+) Idiomático:**
    *   Estructuras asíncronas limpias mediante `async/await` con control riguroso de errores (`try/catch`).
    *   Uso avanzado de destructuración, operador de propagación (*spread operator*), encadenamiento opcional (`?.`) y operador de coalescencia nula (`??`).

---

## 🛠️ Checklist del Desarrollador Frontend Senior

- [ ] ¿El componente está modularizado y tiene menos de 150 líneas de plantilla?
- [ ] ¿He abstraído la lógica de negocio no visual en un *composable* o en una tienda de Pinia?
- [ ] ¿El layout es completamente responsive utilizando CSS Grid / Flexbox y variables CSS?
- [ ] ¿He implementado carga diferida (*lazy loading*) en las rutas principales de la aplicación?
- [ ] ¿Los elementos interactivos son semánticos y accesibles para lectores de pantalla?
- [ ] ¿Se manejan de forma visual los estados de carga (*loading*) y de error al interactuar con el backend?
- [ ] ¿Se limpian adecuadamente todos los timers o suscripciones al desmontar los componentes?
