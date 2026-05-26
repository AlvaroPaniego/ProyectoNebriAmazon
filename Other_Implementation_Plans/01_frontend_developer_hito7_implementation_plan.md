# 🌌 Plan de Implementación de Ingeniería Frontend: Iteración 3 (Catálogo, Búsqueda Avanzada y Detalle)

Este documento detalla el plan técnico para la implementación completa y modular de la **Iteración 3** para **NebriAmazon**, bajo estrictos estándares de **Clean Code**, principios **SOLID**, desacoplamiento total y rendimiento de memoria optimizado mediante una capa de caché en cliente.

---

## 📋 Objetivos de la Iteración

1. **Capa de Red y Datos Desacoplada (SOLID & Dry):**
   - Crear un cliente REST en `productService.js` utilizando Axios para interactuar de forma reactiva con el backend, con un motor de simulación de alta fidelidad robusto ante fallos.
   - Codificar `products.js` en Pinia (`useProductStore`) que centralice el catálogo, filtre los datos reactivamente en el cliente, y mantenga un sistema de caché indexado para evitar peticiones redundantes.

2. **Componentes con Soporte Dinámico y Alto Rendimiento (Atomic Design):**
   - **`SearchBar.vue` (Refactor):** Integrar bidireccionalmente el campo de búsqueda con el store global, respetando al 100% el maquetado CSS premium ya existente.
   - **`ProductCard.vue` (Atoms):** Ficha accesible (WCAG 2.1 AA) e interactiva, con hover zoom suave y navegación por teclado segura.
   - **`ProductGrid.vue` (Organisms):** Grid responsivo que mitigue el Cumulative Layout Shift (CLS) mostrando *Skeleton Loaders* con efecto brillo animado en bucle.

3. **Vistas Core Responsivas y Accesibles:**
   - **`CatalogView.vue`:** Maquetado responsivo con CSS Grid que integre la búsqueda, categorías y rejilla, utilizando `aria-live="polite"` para anunciar a los lectores de pantalla los cambios de conteo.
   - **`ProductDetailView.vue`:** Detalle del producto con carga asíncrona mediante SKU, visor de stock real, selector dinámico de cantidad validado y adición transparente al carrito a través del `useCartStore`.

---

## 🏛️ Propuesta de Diseño Técnico y Arquitectura

### 🧬 Capa de Datos: `productService.js` y `useProductStore` (Pinia)
Para certificar el **Flujo de Datos Unidireccional** y los principios **SOLID**:
- Los componentes y vistas **nunca** consumen Axios de forma directa.
- El servicio de productos expone funciones puras que retornan directamente `response.data`.
- En caso de caída de red o falta de backend, el servicio activa su modo de simulación de forma transparente devolviendo una lista completa de categorías y productos con imágenes premium de alta resolución (Unsplash).
- El Pinia store indexa productos y categorías. Cuenta con una estructura de caché para evitar consultas redundantes:
  ```javascript
  const cache = ref({
    products: null,
    categories: null,
    productBySku: {}
  });
  ```

### 🎨 Componentes Visuales y Estilos Vanilla Scoped
- Uso estricto de variables globales HSL (`variables.css`) para color de marca, bordes y transiciones.
- Estados de enfoque `:focus-visible` perfectamente legibles, con contornos brillantes (`outline: 2px solid var(--color-accent)`) para asegurar la conformidad WCAG 2.1 AA.
- Las vistas utilizarán **CSS Grid** de forma responsiva: `grid-template-columns: repeat(auto-fill, minmax(280px, 1fr))` para un comportamiento natural.

---

## 📂 Archivos a Modificar / Crear

### 🧬 Capa de Datos y Red (Servicios y Store)

#### [NEW] [productService.js](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/services/productService.js)
Servicio asíncrono que realiza peticiones HTTP con Axios (`api.js`). Métodos expuestos:
- `getProducts()`: Retorna `response.data` de `/products`.
- `getProductBySku(sku)`: Retorna `response.data` de `/products/sku/:sku`.
- `getCategories()`: Retorna `response.data` de `/categories`.
- *Simulador Local Integrado:* Si ocurre un error de red o de servidor, el servicio capturará la excepción y retornará de forma transparente un conjunto de productos y categorías de alta fidelidad, asegurando la operabilidad offline de la app.

#### [NEW] [products.js (Store)](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/store/products.js)
Store de Pinia `useProductStore` con:
- *State:* `products`, `categories`, `searchQuery`, `selectedCategory`, `loading`, `error`, `cache`.
- *Getters:* `getFilteredProducts` computado reactivo combinando filtros de texto y categorías.
- *Actions:*
  - `fetchProducts(force = false)`: Llama a `productService.getProducts()`. Si hay datos en caché y no se fuerza la recarga, utiliza la memoria local de forma instantánea.
  - `fetchProductBySku(sku, force = false)`: Busca primero el SKU en el catálogo cargado o en el caché de productos individuales. Si no existe, realiza la petición asíncrona.
  - `fetchCategories(force = false)`: Llama a `productService.getCategories()` utilizando caché local.
  - Mutadores para filtros: `setSearchQuery(query)`, `setSelectedCategory(categoryId)`.

---

### 🎨 Componentes Visuales

#### [MODIFY] [SearchBar.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/components/molecules/SearchBar.vue)
- Modificar **únicamente** la sección `<script setup>` para sincronizar de forma bidireccional el término de búsqueda con la propiedad `searchQuery` de `useProductStore`.
- Al realizar la búsqueda (submit) o al limpiar el input (clear), se actualizará el estado del store de manera transparente para reactivar la computación del getter `getFilteredProducts`.

#### [NEW] [ProductCard.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/components/atoms/ProductCard.vue)
- Tarjeta de producto premium con semántica HTML5 (`<article>`).
- Muestra: imagen con zoom hover, título completo, precio formateado con dos decimales en euros, SKU y etiqueta de disponibilidad en stock (en verde si hay existencias, en rojo si está agotado).
- Botón accesible para ver detalles, optimizado para navegación por teclado mediante `:focus-visible`.

#### [NEW] [ProductGrid.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/components/organisms/ProductGrid.vue)
- Rejilla responsiva construida con CSS Grid.
- Si `loading === true`, renderizará en bucle 6 *Skeleton Loaders* animados con efecto degradado dinámico (shimmer).
- Si el listado de productos filtrados está vacío, muestra un mensaje descriptivo de "Sin resultados".
- Si hay artículos, renderiza `ProductCard.vue` asignando identificadores estables (`:key="product.id"`).

---

### 🚀 Vistas e Integración de Enrutamiento

#### [NEW] [CatalogView.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/views/CatalogView.vue)
- Página general del catálogo con layout estructurado en una barra lateral de categorías y un grid central.
- Dispara `fetchProducts()` y `fetchCategories()` en el ciclo `onMounted`.
- Utiliza la directiva `aria-live="polite"` para anunciar reactivamente el total de artículos encontrados.
- Diseño responsivo: se oculta la barra lateral en pantallas pequeñas pasando a un selector móvil y grid de una sola columna.

#### [NEW] [ProductDetailView.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/views/ProductDetailView.vue)
- Vista detallada del producto. Recupera el SKU dinámico desde la ruta (`route.params.sku`).
- Muestra el stock real, precio de precisión, descripción completa y ficha técnica.
- Selector de cantidad reactivo con validaciones preventivas:
  - No puede superar el stock real del producto en tienda.
  - No puede ser inferior a 1.
- Botón premium con loader reactivo que ejecuta la acción `addItem(product, quantity)` de `useCartStore` de forma asíncrona.

---

## 🧪 Plan de Verificación

### Pruebas Unitarias y de Reactividad (Vitest)
- Probar que `useProductStore` filtra correctamente la colección al cambiar `searchQuery` o `selectedCategory`.
- Verificar que la capa de caché de `useProductStore` funciona correctamente y no se realizan llamadas múltiples para el mismo recurso.

### Verificación Manual de Accesibilidad (WCAG 2.1 AA)
- Verificar mediante navegación con teclado (`Tab` y `Enter`) que todos los elementos activos (botones, inputs, tarjetas) reciben el estado de foco y son operables.
- Validar contraste cromático mediante herramientas integradas de Chrome/Firefox en modo oscuro y claro.
- Comprobar que los lectores de pantalla anuncian correctamente la carga y el conteo de elementos con `aria-live`.
