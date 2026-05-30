# Plan de Implementación: Componente de Filtros por Categorías e Historial de Pedidos 🛍️

Este documento describe el plan técnico detallado para implementar el componente reutilizable de filtros por categorías y la vista interactiva del historial de pedidos del usuario, conforme a la arquitectura de **Atomic Design** y **Pinia** definidos para **NebriAmazon**.

---

## 🏛️ Decisiones de Arquitectura e Integración

1. **Estructura SOLID y Clean Code**:
   - **Single Responsibility**: Crearemos un store de Pinia separado para órdenes (`src/store/orders.js`) en lugar de acoplarlo con el carrito (`src/store/cart.js`). Así, la lógica del carrito (alta frecuencia de mutaciones en local/remoto) queda desacoplada de la consulta de pedidos históricos.
   - **Desacoplamiento de Red**: Toda llamada a Axios se encapsulará en `src/services/orderService.js` y `src/services/productService.js`. Los stores de Pinia gestionarán los estados reactivos, invocando estos servicios.
   
2. **Resiliencia ante Fallos de API (Desarrollo Visual Autónomo)**:
   - Implementaremos bloques `try/catch` rigurosos en las acciones de los stores.
   - Si las peticiones de red fallan (debido a que el backend no está iniciado o devuelve un 404):
     - El store de productos cargará una colección de **categorías fallback** de alta calidad y un catálogo inicial con imágenes, permitiendo que la interfaz siga funcionando visualmente.
     - El store de órdenes mantendrá inicialmente un estado vacío de órdenes para validar el estado sin pedidos en la interfaz de usuario, pero permitirá cargar **órdenes fallback** de prueba en caso de fallo para inspeccionar visualmente su diseño.

3. **Alineación con HSL y Atomic Design**:
   - Crearemos la molécula `CategoryFilter.vue` en `src/components/molecules/`. Este componente extraerá y encapsulará la representación en chips horizontales (móvil) y lista con viñetas (escritorio) utilizando las variables HSL de `variables.css`.
   - Modificaremos `CatalogView.vue` para reemplazar el HTML embebido por este componente reutilizable.
   - Soportaremos navegación accesible por teclado (con `tabindex`, `focus-visible`, etc.) de acuerdo con la norma WCAG 2.1 AA.

---

## 📂 Propuesta de Cambios por Componente

### 📦 1. Capa de Servicios

#### [MODIFY] [orderService.js](file:///d:/Inst_Nebrija_2025/CLASES_IABD_(G1)/Programación_de_IA/Prof_Rubén/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/services/orderService.js)
- Añadir la función asíncrona `getOrders` para realizar la petición `GET /orders` (la cual se convertirá a `/api/orders` en virtud de la configuración de base URL e interceptores de Axios).
```javascript
export const getOrders = async () => {
  const response = await api.get('/orders');
  return response.data;
};
```

---

### 🍍 2. Capa de Estado (Pinia Stores)

#### [MODIFY] [products.js](file:///d:/Inst_Nebrija_2025/CLASES_IABD_(G1)/Programación_de_IA/Prof_Rubén/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/store/products.js)
- Asegurar que el getter reactivo esté expuesto con el nombre `filteredProducts` (además de mantener o aliasar `getFilteredProducts` para no romper llamadas existentes).
- En el bloque `catch` de `fetchCategories`, si ocurre un error, cargar categorías de fallback en memoria:
  ```javascript
  const DEFAULT_CATEGORIES = [
    { id: 1, name: 'Electrónica' },
    { id: 2, name: 'Libros' },
    { id: 3, name: 'Hogar y Cocina' },
    { id: 4, name: 'Ropa y Accesorios' },
    { id: 5, name: 'Deportes' }
  ];
  ```
- En el bloque `catch` de `fetchProducts`, cargar productos de fallback realistas con sus respectivas propiedades y asociados a estas categorías.

#### [NEW] [orders.js](file:///d:/Inst_Nebrija_2025/CLASES_IABD_(G1)/Programación_de_IA/Prof_Rubén/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/store/orders.js)
- Inicializar el estado de órdenes:
  - `orders: []` (Listado de órdenes históricas del usuario).
  - `loading: false` (Indicador de carga asíncrona).
  - `error: null` (Mensaje de error en caso de fallo crítico).
- Implementar la acción `fetchOrders` con manejo asíncrono y protección try/catch:
  - En caso de éxito, actualizar el array de órdenes con la respuesta de la API.
  - En caso de error (red o 404), implementar un fallback dinámico controlado: si el usuario está autenticado y la API falla, cargar una lista de órdenes simuladas realistas con estados `'paid'` y `'pending'` y precios exactos para garantizar que la UI se pueda depurar visualmente.

---

### 🎨 3. Componentes de Interfaz de Usuario (Atomic Design)

#### [NEW] [CategoryFilter.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_(G1)/Programación_de_IA/Prof_Rubén/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/components/molecules/CategoryFilter.vue)
- Componente que consume reactivamente el `useProductStore`.
- Renderiza el selector de categorías adaptativo:
  - **Móvil/Tablet**: Contenedor horizontal scrollable con chips interactivos (`.mobile-categories-chips`).
  - **Escritorio**: Barra lateral vertical con viñetas estilizadas y transiciones de escala al pasar el ratón (`.categories-list`).
- Cumple rigurosamente con los tokens HSL (`--color-accent`, `--color-primary`, `--transition-fast`) y los atributos de accesibilidad WCAG (`aria-label`, `tabindex`, `role="button"`).

#### [MODIFY] [CatalogView.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_(G1)/Programación_de_IA/Prof_Rubén/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/views/CatalogView.vue)
- Importar y renderizar el componente modular `CategoryFilter` reemplazando los bloques duplicados de visualización de categorías, delegándole el control reactivo del filtrado.

#### [NEW] [OrdersView.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_(G1)/Programación_de_IA/Prof_Rubén/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/views/OrdersView.vue)
- Vista completa del historial de pedidos del cliente, accesible tras loguearse.
- **Estado de Carga (Skeleton/Spinner)**: Muestra retroalimentación visual animada mientras `loading === true`.
- **Estado Vacío (No Orders)**:
  - Renderiza una ilustración o caja de diseño premium (glassmorphism, bordes sutiles).
  - Muestra el texto "Aún no tienes pedidos" y un botón descriptivo "Explorar catálogo" que redirige al usuario a la vista de catálogo.
- **Estado con Datos (Lista de Órdenes)**:
  - Renderiza una lista estructurada semánticamente con `<article>` por cada orden.
  - Para cada pedido, muestra de manera clara y premium:
    - ID del pedido destacado (ej. `#NA-X90A1`).
    - Fecha de creación formateada en español.
    - Código de seguimiento (`tracking_code`) copiable o destacado.
    - Estado de pago (`status`) mediante un **Badge Dinámico**:
      - `paid`: Color verde de éxito (`--color-success`) con animación sutil.
      - `pending`: Color amarillo/naranja de advertencia (`--color-warning`).
    - Desglose de artículos comprados en la orden (nombre del producto, cantidad, precio unitario).
    - Importe total calculado y renderizado de forma clara (ej. `145.20 €`).

---

### 🌐 4. Rutas y Navegación SPA

#### [MODIFY] [index.js](file:///d:/Inst_Nebrija_2025/CLASES_IABD_(G1)/Programación_de_IA/Prof_Rubén/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/router/index.js)
- Añadir la ruta `/orders` asociada a `OrdersView.vue` con soporte de carga diferida (lazy loading).
- Asegurar que la ruta tenga metadatos de protección: `meta: { requiresAuth: true }`. Esto impedirá que usuarios anónimos accedan a sus pedidos, redirigiéndolos al login y preservando la redirección en query params.

#### [MODIFY] [AppNavbar.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_(G1)/Programación_de_IA/Prof_Rubén/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/components/organisms/AppNavbar.vue)
- Habilitar en `handleNavigation` el redireccionamiento para el caso `'orders'` apuntando a la ruta `{ name: 'Orders' }`.

---

## 🧪 Plan de Verificación

### Pruebas Unitarias e Integración (Vitest)
- Crearemos o adaptaremos pruebas unitarias de los almacenes para asegurar la reactividad.
- Probaremos que al actualizar `selectedCategory` en el store de productos, el getter `filteredProducts` filtre correctamente los elementos.
- Validaremos que el store de órdenes inicialice con un array vacío y que su comportamiento asíncrono tolere errores de la API.

### Verificación Manual de la Interfaz
1. **Verificación de Filtros**:
   - Abrir el catálogo e interactuar con los chips de categorías en tamaño móvil y con la barra lateral en tamaño escritorio.
   - Confirmar que al pulsar una categoría, los productos cambien reactivamente y de forma instantánea.
   - Comprobar que los estilos utilicen los tonos de `--color-accent` y respondan al focus del teclado.

2. **Verificación del Historial de Pedidos**:
   - Iniciar sesión en la aplicación.
   - Navegar al historial de pedidos a través del enlace de la barra de navegación "Devoluciones & Pedidos".
   - Detener el servidor backend para forzar un error de red y validar la resiliencia: la UI debe cargar los datos del fallback de forma impecable sin lanzar errores en blanco.
   - Realizar compras simuladas en el checkout y validar que al completarse se agreguen y rendericen en el listado de pedidos.
