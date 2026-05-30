# Plan de Implementación: Historial y Estado de Pedidos del Usuario

Este plan detalla los pasos para diseñar, estructurar e implementar la funcionalidad del **Historial y Estado de Pedidos** (Hito Extra) en el frontend de NebriAmazon. Se priorizarán los estándares de **Clean Code** de la skill asignada: nombres legibles y auto-descriptivos, desacoplamiento absoluto de lógica y presentación, semántica accesible (WCAG 2.1 AA), resiliencia de red y una interfaz de usuario fluida con micro-interacciones premium usando variables HSL globales.

---

## Modificaciones Propuestas

### 1. Servicio de Pedidos: `src/services/orderService.js`
Añadiremos la función asíncrona faltante en el servicio `orderService.js` para asegurar que el store invoque el endpoint real del backend `/orders`.

#### [MODIFY] [orderService.js](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/services/orderService.js)
- Añadir la función exportada asíncrona `getUserOrders()` que realiza una llamada `api.get('/orders')` y retorna `response.data`.
- Capturar y propagar de forma consistente cualquier error tipado o de red.

---

### 2. Store de Pinia: `src/store/orders.js`
Crearemos un store modular de Pinia siguiendo el estándar del proyecto (Composition API).

#### [NEW] [orders.js](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/store/orders.js)
- **Estado (State):**
  - `orders` (`ref([])`): Lista de pedidos del usuario autenticado.
  - `loading` (`ref(false)`): Booleano para el estado de carga (skeleton screen).
  - `error` (`ref(null)`): Objeto de error o string para fallos de red.
- **Acciones (Actions):**
  - `fetchUserOrders()`: Función asíncrona que:
    - Reinicia `error.value = null` y establece `loading.value = true`.
    - Llama a `getUserOrders()` del servicio de forma segura.
    - Asigna los datos obtenidos a `orders.value` (ordenados en el backend por `created_at desc`, pero con reaseguro en el frontend).
    - Captura limpiamente excepciones de red y asigna un mensaje de error amigable o técnico controlado (ej. "Error de red al conectar con el servidor" o el mensaje directo del backend).
    - Asegura que `loading.value = false` en el bloque `finally`.

---

### 3. Enrutamiento del Frontend: `src/router/index.js`
Registraremos la ruta `/orders` de forma modular.

#### [MODIFY] [index.js](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/router/index.js)
- Registrar la ruta `/orders` bajo el nombre `OrdersHistory` cargando dinámicamente (`lazy-loading`) `@/views/OrdersHistoryView.vue`.
- Asegurar que la redirección a `/login` ocurra limpiamente si se intenta acceder sin sesión activa.

---

### 4. Navegación Principal: `src/components/organisms/AppNavbar.vue`
Conectaremos la navegación existente con la nueva vista.

#### [MODIFY] [AppNavbar.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/components/organisms/AppNavbar.vue)
- Actualizar la lógica del método `handleNavigation` para que al recibir la acción `'orders'`, realice un `router.push({ name: 'OrdersHistory' })` en lugar de ignorarla.

---

### 5. Vista de la Interfaz: `src/views/OrdersHistoryView.vue`
Diseñaremos una página de visualización Premium y accesible adaptada a móviles y escritorio.

#### [NEW] [OrdersHistoryView.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/views/OrdersHistoryView.vue)
- **Estructura Reactiva (`<script setup>`):**
  - Conexión al store de órdenes (`useOrdersStore()`) mediante `storeToRefs`.
  - Conexión paralela al store de productos (`useProductStore()`) para precargar el catálogo y mapear imágenes/nombres en tiempo real sin requerir mock data.
  - Formateadores de fecha localizados (`Intl.DateTimeFormat`) y formateadores de moneda (`Intl.NumberFormat`).
  - Lógica para expandir/colapsar el desglose de productos de cada pedido de forma individual.
- **Visual A (Carga / Shimmer Loading):**
  - Skeleton screens animados usando variables CSS nativas que simulan la estructura exacta de las tarjetas de pedidos para mitigar el Cumulative Layout Shift (CLS).
- **Visual B (Vacío Inteligente / Error de red):**
  - Contenedor elegante de vacío ilustrado.
  - Si hay un error de red o `orders.length === 0`, muestra un botón "Seguir Comprando" que redirige a `/catalog`.
- **Visual C (Lista de Pedidos):**
  - Tarjetas elegantes ordenadas decrecientemente por fecha.
  - **Estado del Pedido (Badge HSL):** Badge sutil reactivo al estado:
    - *Entregado*: Fondo verde claro con opacidad `hsla(142, 60%, 30%, 0.1)`, texto `hsl(142, 60%, 30%)`.
    - *Enviado*: Fondo azul claro con opacidad `hsla(217, 91%, 60%, 0.1)`, texto `hsl(217, 91%, 60%)`.
    - *Procesando / Pendiente*: Fondo amarillo/naranja claro con opacidad `hsla(36, 100%, 50%, 0.15)`, texto `hsl(27, 98%, 53%)`.
  - **Desglose de productos colapsable:**
    - Cabecera interactiva con un botón que cambia de estado (abierto/cerrado) de forma accesible (`aria-expanded`).
    - Icono chevron indicador con animación de rotación fluida mediante transiciones CSS.
    - Renderización de cada ítem: Imagen miniatura (resolución a través del store de productos), nombre del producto, cantidad y coste unitario/total formateado.
- **Alineación con el Sistema de Diseño:**
  - Estilos scoped usando de forma estricta las variables HSL globales como `--color-surface`, `--color-accent`, `--font-sans`, y `--color-text-muted`.

---

## Plan de Verificación

### Pruebas Manuales
1. **Flujo de Acceso:**
   - Iniciar sesión en la plataforma y hacer clic en "Devoluciones & Pedidos". Verificar la carga y la visualización de la lista.
2. **Estados Visuales:**
   - Forzar el estado de carga (simulando red lenta en DevTools) para validar el efecto *shimmer* y la prevención del CLS.
   - Verificar el estado vacío realizando una cuenta nueva sin compras.
   - Simular un corte de conexión a internet o apagar temporalmente el backend para validar el manejo del "Network Error" y el botón para volver al catálogo.
3. **Mapeo de Productos:**
   - Confirmar que los nombres e imágenes de los productos en el desglose colapsable correspondan perfectamente a los del catálogo real gracias al acoplamiento dinámico con `useProductStore`.
4. **Diseño Responsivo:**
   - Probar en resoluciones móviles y escritorio usando DevTools para garantizar la legibilidad y la fluidez de las micro-animaciones.
