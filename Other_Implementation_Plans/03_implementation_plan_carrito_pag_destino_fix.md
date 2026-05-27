# Plan de Implementación: Gestión y Vista de Carrito de Compras (CartView)

Este plan detalla los pasos para crear la vista real y funcional del Carrito de Compras (`CartView.vue`) en `frontend-nebri-amazon`, solucionar el enrutamiento del Navbar e integrar la tienda de Pinia (`useCartStore`) de manera reactiva para ofrecer una experiencia interactiva y premium inspirada en Amazon.

## User Review Required

> [!IMPORTANT]
> - **Visual del Carrito (Amazon-Style):** La interfaz se estructurará en dos columnas: listado detallado de productos con controles de cantidad y botón de eliminación a la izquierda, y la tarjeta de desglose financiero (Subtotal, impuestos, envío gratis) con el botón "Proceder al pago" en color dorado de acento a la derecha.
> - **Actualización en AppNavbar:** Modificaremos la lógica del Navbar para que la redirección sea robusta y directa utilizando `useRouter` internamente, además de emitir el evento `@navigate`, asegurando que el clic en el carrito funcione desde cualquier página.
> - **Conexión de Flujo de Pago:** Añadiremos la ruta `/checkout` (para `CheckoutView.vue`) al archivo de enrutamiento principal para que el botón "Proceder al pago" del carrito funcione de forma real.

## proposed-changes

---

### [Enrutamiento]

Añadiremos tanto la ruta `/cart` como `/checkout` para permitir la navegación completa del flujo de compra.

#### [MODIFY] [index.js](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/router/index.js)
- Importar `CartView.vue` y `CheckoutView.vue` de forma estática o dinámica.
- Declarar la ruta `/cart` con el nombre `'Cart'`.
- Declarar la ruta `/checkout` con el nombre `'Checkout'`.

---

### [Cabecera Global]

Haremos que la barra de navegación redirija directamente a las rutas correctas usando el enrutador en lugar de depender únicamente de controladores repetidos en los componentes padres.

#### [MODIFY] [AppNavbar.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/components/organisms/AppNavbar.vue)
- Importar `useRouter` de `vue-router`.
- Actualizar `handleNavigation` para realizar navegación programática a `'Home'`, `'Login'`, `'Catalog'` y `'Cart'` según el destino seleccionado.

---

### [Vistas y Controladores de Página]

#### [MODIFY] [HomeView.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/views/HomeView.vue)
- Añadir el controlador para la navegación del carrito (`'cart'`) en `handleNavigation` como medida de seguridad y consistencia visual.

#### [MODIFY] [LoginView.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/views/LoginView.vue)
- Actualizar `handleNavigation` para mapear `'cart'` hacia el enrutador de forma real (`router.push({ name: 'Cart' })`).

#### [MODIFY] [RegisterView.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/views/RegisterView.vue)
- Actualizar `handleNavigation` para mapear `'cart'` hacia el enrutador de forma real (`router.push({ name: 'Cart' })`).

#### [NEW] [CartView.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/views/CartView.vue)
- Integrar `AppNavbar` y `ChatbotWidget`.
- Vincular reactivamente el estado global de `cartStore` (`items`, `loading`, `totalItemsCount`, `subtotal`, `taxAmount`, `total`).
- **Diseño Premium de Dos Columnas:**
  - **Izquierda:** Listado de productos del carrito con imagen del producto, título, precio, selector de cantidades dinámico (hasta el límite de stock disponible) y botón de "Eliminar" estilizado.
  - Mostrar la sección ilustrativa si el carrito está vacío: mensaje *"Tu carrito de NebriAmazon está vacío"* y un botón llamativo de *"Volver a comprar"*.
  - **Derecha:** Tarjeta de Subtotal con cálculo preciso, indicador de "Envío Gratis" y botón color oro/amarillo premium "Proceder al pago" conectado a la ruta `/checkout`.
- Animaciones fluidas al añadir/eliminar elementos (`fade-out/fade-in` de elementos de la lista).

## Verification Plan

### Automated / Manual Verification
1. **Comportamiento del Navbar:**
   - Clicar en el botón "Carrito" del Navbar en la Home, Login o Registro y verificar la redirección a `/cart`.
   - Verificar que el contador flotante (BaseBadge) sobre el icono del carrito refleje con total precisión la cantidad acumulada de productos.
2. **Interactividad del Carrito:**
   - Añadir un producto en Home y comprobar que se actualiza reactivamente el carrito al navegar a `/cart`.
   - Cambiar la cantidad del producto mediante el selector y verificar que los subtotales, IVA y total de la tarjeta derecha se recalculan instantáneamente.
   - Tratar de seleccionar una cantidad mayor al stock disponible (el componente debe limitar las opciones o mostrar un aviso controlado).
   - Pulsar en "Eliminar" y verificar la desaparición del artículo con una transición suave.
3. **Flujo de Pago:**
   - Pulsar en "Proceder al pago" con productos en el carrito y comprobar que redirige correctamente a `/checkout` para rellenar los datos de envío y facturación.
