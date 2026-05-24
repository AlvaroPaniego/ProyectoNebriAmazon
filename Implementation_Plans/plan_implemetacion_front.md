# 🌌 Plan de Implementación de Ingeniería Frontend: NebriAmazon 🛍️

Este documento representa el **Plan de Implementación de Ingeniería del Lado del Cliente (Frontend)** para **NebriAmazon**, derivado de las especificaciones maestras del *PRD_Global_NebriAmazon.md*. Establece la arquitectura de software, el diseño de componentes bajo los principios de *Atomic Design*, la gestión reactiva de estado con *Pinia*, y las directrices de estilo utilizando *Vanilla CSS* premium, sirviendo como la guía técnica definitiva para los agentes `frontend-developer`, `frontend-tester` y `code-inspector`.

---

## 🏛️ 1. Arquitectura de Software y Pila Tecnológica

La aplicación frontend se construirá como una **Single Page Application (SPA)** de alto rendimiento, optimizada para ofrecer una experiencia de usuario fluida a 60fps con transiciones y micro-animaciones de calidad premium.

### 🛠️ Core Stack del Frontend
*   **Framework base:** Vue 3.5+ (utilizando exclusivamente **Composition API** con la sintaxis `<script setup>`).
*   **Gestión de Estado Global:** Pinia (tiendas modulares estructuradas por dominio).
*   **Enrutamiento SPA:** Vue Router con estrategias de carga diferida (*lazy loading*).
*   **Compilador y Herramienta de Construcción:** Vite (configuración optimizada con alias de rutas `@` para `/src`).
*   **Capa de Estilos:** Vanilla CSS moderno (aprovechando variables CSS personalizadas, Flexbox, y CSS Grid). Queda explícitamente descartado el uso de TailwindCSS u otros frameworks de utilidades para garantizar un control estético total y un bundle ultraligero.
*   **Suite de Pruebas:** Vitest para pruebas unitarias de stores y lógica financiera de precisión, y Vue Test Utils para pruebas de integración de componentes visuales.
*   **Cliente HTTP:** Axios para comunicación asíncrona robusta y gestión unificada de cabeceras JWT mediante interceptores.

---

## 📂 2. Estructura de Directorios y Diseño Atómico (Atomic Design)

Para maximizar la reusabilidad, aislamiento y mantenibilidad del software, el directorio principal de la aplicación `frontend-nebri-amazon/` adoptará la siguiente estructura estandarizada basada en **Atomic Design**:

```text
frontend-nebri-amazon/
├── public/                 # Assets estáticos globales (favicons, logos del sistema)
├── src/
│   ├── assets/             # Variables CSS globales, estilos base y temas HSL
│   │   ├── main.css        # Punto de entrada de CSS e importaciones de variables
│   │   └── variables.css   # Sistema de diseño con tokens de color, sombras y tipografías
│   ├── components/
│   │   ├── atoms/          # Componentes mínimos con alta cohesión y sin estado de negocio directo
│   │   │   ├── BaseButton.vue  # Botón dinámico con loader rotativo integrado
│   │   │   ├── BaseInput.vue   # Campo de formulario interactivo con validación visual en tiempo real
│   │   │   ├── BaseBadge.vue   # Burbuja flotante con animación "pulse" para contador del carrito
│   │   │   └── BaseRating.vue  # Visualizador reactivo de valoración con estrellas
│   │   ├── molecules/      # Agrupación de átomos con estado acotado
│   │   │   ├── ProductCard.vue # Ficha del catálogo con imagen, estrellas, precio y botón de añadir
│   │   │   ├── CartItem.vue    # Fila de producto en el carrito con selector de cantidad interactivo
│   │   │   └── SearchBar.vue   # Barra de búsqueda con autocompletado y debounce reactivo
│   │   └── organisms/      # Módulos complejos con lógica de negocio integrada
│   │       ├── AppNavbar.vue   # Cabecera responsiva con controles de perfil, buscador y cesta
│   │       ├── ProductGrid.vue # Contenedor de rejilla responsiva que gestiona el catálogo filtrado
│   │       ├── CheckoutForm.vue# Formulario complejo de recopilación de envíos y pagos simulados
│   │       └── ChatbotWidget.vue# Widget de soporte pasivo flotante con comunicación asíncrona
│   ├── services/           # Clientes HTTP y llamadas desacopladas de red
│   │   ├── api.js          # Configuración base de Axios e interceptores JWT
│   │   ├── authService.js  # Peticiones de registro, login y perfil
│   │   ├── productService.js# Endpoints de consulta y CRUD de productos
│   │   ├── cartService.js  # Sincronización asíncrona del carrito con la base de datos
│   │   ├── orderService.js # Procesamiento de pedidos y checkout
│   │   └── chatbotService.js# Interacción de lenguaje natural con el backend
│   ├── store/              # Tiendas modulares de Pinia
│   │   ├── auth.js         # Estado de la sesión, token JWT y perfil de usuario
│   │   ├── products.js     # Estado del catálogo, términos de búsqueda y filtros
│   │   └── cart.js         # Estado reactivo de la cesta con precisión financiera
│   ├── router/             # Configuración de Vue Router y Navigation Guards
│   │   └── index.js
│   ├── views/              # Vistas principales cargadas mediante Lazy Loading
│   │   ├── HomeView.vue        # Landing Page con ofertas destacadas y accesos directos
│   │   ├── CatalogView.vue     # Panel de catálogo general con filtros laterales
│   │   ├── ProductDetailView.vue# Ficha detallada del producto con control de stock y SKU
│   │   ├── CartView.vue        # Vista del desglose y gestión del carrito
│   │   ├── CheckoutView.vue    # Pasarela de confirmación de compra y pago simulado
│   │   ├── LoginView.vue       # Acceso seguro a la plataforma
│   │   ├── RegisterView.vue    # Registro de nuevos usuarios
│   │   └── AdminDashboardView.vue # Consola de administración para la gestión de productos (CRUD)
│   ├── App.vue             # Componente raíz del layout global
│   └── main.js             # Punto de entrada de montaje de la aplicación
├── index.html              # HTML5 estructurado semánticamente y optimizado para SEO
├── package.json            # Dependencias y scripts del proyecto
├── vite.config.js          # Configuración de compilación, alias `@` y dev-server
└── vitest.config.js        # Configuración del entorno de pruebas unitarias
```

---

## 🍍 3. Arquitectura del Estado Reactivo (Pinia Stores)

Para evitar la sobrecarga de consultas a la red y estructurar un flujo de datos unidireccional predecible, se implementarán tres tiendas (*stores*) principales en Pinia:

### 3.1. `useAuthStore` (Gestión de Sesión)
*   **State:**
    *   `token`: String JWT. Persistido automáticamente de forma segura en `localStorage`.
    *   `user`: Objeto con las propiedades del usuario logueado (`id`, `name`, `email`, `role`).
    *   `isAuthenticated`: Booleano deducido reactivamente del estado del token.
    *   `loading`: Booleano para deshabilitar interacciones durante peticiones de login/registro.
*   **Getters:**
    *   `isAdmin`: Retorna `true` si `user.role === 'admin'`.
*   **Actions:**
    *   `login(credentials)`: Envía credenciales al backend, almacena el token JWT, inicializa el perfil en el estado y sincroniza el carrito local con el backend.
    *   `register(userData)`: Registra un nuevo usuario y realiza el login automático.
    *   `logout()`: Limpia el token y el perfil de usuario del estado y de `localStorage`, vacía las tiendas de carrito y productos, y redirige al catálogo.
    *   `fetchCurrentUser()`: Recupera la información del usuario (`/api/auth/me`) si hay un token persistido al recargar la aplicación.

### 3.2. `useProductStore` (Gestión del Catálogo)
*   **State:**
    *   `products`: Lista completa de productos cargados.
    *   `selectedProduct`: Objeto con la información detallada de un producto seleccionado.
    *   `categories`: Listado de categorías disponibles para filtrado.
    *   `filters`: Objeto con los filtros activos (`category_id`, `searchQuery`, `minPrice`, `maxPrice`).
    *   `loading`: Estado de carga para el esqueleto (*skeleton*) visual del catálogo.
*   **Getters:**
    *   `filteredProducts`: Retorna los productos aplicando de forma reactiva los filtros en el cliente (para una respuesta instantánea al escribir o cambiar filtros).
*   **Actions:**
    *   `fetchProducts(params)`: Solicita los productos al servidor utilizando los query params requeridos.
    *   `fetchProductById(id)`: Solicita el detalle de un único producto.
    *   `createProduct(productData)`: (Solo Admin) Llama a la API para crear un producto e incrementa localmente la lista.
    *   `updateProduct(id, productData)`: (Solo Admin) Modifica el producto en base de datos e integra los cambios reactivamente.
    *   `deleteProduct(id)`: (Solo Admin) Realiza el borrado lógico en backend (Soft Delete) y remueve el producto del catálogo en pantalla.

### 3.3. `useCartStore` (Cálculo Financiero y Sincronización)
Para evitar discrepancias de céntimos en operaciones aritméticas binarias de punto flotante en JavaScript, todos los cálculos financieros implementarán getters que operen con escalado entero o conversiones matemáticas de alta precisión (`toFixed(2)` tras operaciones aritméticas).

*   **State:**
    *   `cartId`: UUID del carrito persistido.
    *   `items`: Array de objetos en la cesta. Formato: `[ { id, quantity, product: { id, name, price, stock, sku, image_urls } } ]`.
    *   `loading`: Booleano para spinners de actualización.
*   **Getters:**
    *   `totalItemsCount`: Sumatorio entero del total de cantidades (`quantity`) añadidas al carrito. Representado en la burbuja `BaseBadge.vue` del Navbar.
    *   `subtotal`: Sumatorio de `producto.price * quantity`.
    *   `taxAmount`: Cálculo del IVA del 21% aplicado sobre el subtotal (`subtotal * 0.21`).
    *   `total`: Suma exacta del subtotal y del IVA (`subtotal + taxAmount` o `subtotal * 1.21`).
*   **Actions:**
    *   `initializeCart()`: Carga el carrito desde el almacenamiento local (`localStorage`) si el usuario está anónimo. Si el usuario se autentica, realiza una petición de sincronización con el servidor (`/api/cart`) combinando los ítems locales con los de la base de datos de manera inteligente.
    *   `addItem(product, quantity = 1)`: Valida primero el stock disponible del producto. Añade localmente el ítem y, si está autenticado, despacha la petición `POST /api/cart/items` al servidor.
    *   `updateItemQuantity(itemId, newQuantity)`: Valida los límites de stock. Modifica localmente la cantidad y, si está autenticado, despacha `PUT /api/cart/items/:id`.
    *   `removeItem(itemId)`: Remueve localmente el ítem con transición CSS animada y ejecuta `DELETE /api/cart/items/:id` si corresponde.
    *   `clearCart()`: Vacía por completo la cesta (local y remota) tras completar una orden de compra exitosa.

---

## 🎨 4. Estética de Diseño Premium, Micro-Animaciones y Accesibilidad

NebriAmazon destacará visualmente a través de un diseño vibrante, fluido y profundamente pulido, emulando la jerarquía visual de Amazon bajo un diseño moderno de alta gama (Glassmorphism sutil y transiciones orgánicas).

### 4.1. Tokens de Diseño y Variables CSS (`src/assets/variables.css`)
Se implementará una arquitectura basada en variables HSL dinámicas para facilitar futuras integraciones de temas:

```css
:root {
  /* Paleta de Colores de Marca (NebriAmazon Core) */
  --color-primary: hsl(220, 27%, 10%);       /* Azul Oscuro Premium (#131921) */
  --color-primary-light: hsl(220, 27%, 16%); /* Azul Navbar secundario */
  --color-accent: hsl(36, 100%, 50%);         /* Naranja de Acento Amazon (#FF9900) */
  --color-accent-hover: hsl(36, 100%, 45%);   /* Naranja activo */
  --color-background: hsl(0, 0%, 97%);       /* Gris ultra claro de fondo */
  --color-surface: hsl(0, 0%, 100%);         /* Blanco puro de tarjetas */
  --color-text: hsl(220, 20%, 15%);          /* Negro carbón de legibilidad */
  --color-text-muted: hsl(220, 10%, 50%);    /* Gris intermedio para textos auxiliares */
  --color-border: hsl(220, 12%, 88%);        /* Bordes sutiles y limpios */
  
  /* Estados */
  --color-success: hsl(142, 70%, 45%);       /* Verde stock / confirmación */
  --color-error: hsl(0, 84%, 60%);           /* Rojo alertas / stock agotado */
  --color-warning: hsl(45, 93%, 47%);        /* Amarillo advertencias */
  
  /* Tipografías y Sombras */
  --font-sans: 'Inter', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  --shadow-sm: 0 2px 4px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.08);
  --shadow-lg: 0 10px 25px rgba(0, 0, 0, 0.12);
  
  /* Transiciones Estándar */
  --transition-fast: 0.15s cubic-bezier(0.4, 0, 0.2, 1);
  --transition-normal: 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}
```

### 4.2. Micro-Animaciones Reactivas
1.  **Interactividad en Botones (`BaseButton.vue`):** Efecto hover con traslación vertical sutil `transform: translateY(-2px)` combinada con un ligero aumento de sombra. Cuando se activa el estado de carga (`loading === true`), una transición CSS suave reemplaza el texto por un spinner vectorial rotativo (`@keyframes spin { 100% { transform: rotate(360deg); } }`).
2.  **Transiciones en Tarjetas (`ProductCard.vue`):** Enfoque progresivo (*zoom* suave del 1.05x en la imagen del producto) al pasar el ratón por encima, manteniendo la imagen dentro del contenedor mediante `overflow: hidden`.
3.  **Animación de Cesta (`BaseBadge.vue`):** Efecto de rebote y escala (*pulse-scale*) que se dispara a través de clases dinámicas controladas por el ciclo de vida de Vue cada vez que la cantidad total de la cesta se actualice:
    ```css
    .badge-pop {
      animation: pop 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
    }
    @keyframes pop {
      0% { transform: scale(1); }
      50% { transform: scale(1.4); }
      100% { transform: scale(1); }
    }
    ```
4.  **Skeleton Loading:** Se estructurarán plantillas con un gradiente animado en bucle (`@keyframes shimmer`) para simular la carga de tarjetas de productos mientras `loading === true`, reduciendo el tiempo de carga percibido por el usuario.

### 4.3. Accesibilidad y Estructura Semántica (WCAG 2.1 AA)
Para certificar la accesibilidad web en toda la interfaz:
*   **Semántica Estricta:** Uso estricto de elementos estructurales nativos de HTML5 en la plantilla raíz y vistas (`<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<footer>`).
*   **Atributos ARIA:** Los elementos interactivos personalizados que no utilicen botones nativos incluirán `role="button"`, `aria-label` descriptivos, y soporte para navegación por teclado mediante `tabindex="0"`.
*   **Contraste de Color:** El tamaño del texto y las combinaciones cromáticas de HSL garantizan un contraste superior a **4.5:1** para texto normal y **3:1** para texto grande, cumpliendo holgadamente el estándar AA.
*   **Imágenes Accesibles:** Se forzará a nivel de desarrollo la presencia del atributo descriptivo `alt` en cada etiqueta `<img>`.

---

## 🔗 5. Capa de Servicios y Consumo de API (Integración REST)

Se implementará un cliente Axios centralizado en `src/services/api.js` que configure automáticamente los timeouts y gestione la inyección sistemática de cabeceras de autorización de la siguiente manera:

```javascript
import axios from 'axios';
import { useAuthStore } from '@/store/auth';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:3000/api',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Interceptor de Peticiones: Adjunta JWT Bearer Token
api.interceptors.request.use((config) => {
  const authStore = useAuthStore();
  if (authStore.token) {
    config.headers.Authorization = `Bearer ${authStore.token}`;
  }
  return config;
}, (error) => {
  return Promise.reject(error);
});

// Interceptor de Respuestas: Captura errores comunes (401, 403, 500)
api.interceptors.response.use(
  (response) => response,
  (error) => {
    const authStore = useAuthStore();
    if (error.response && error.response.status === 401) {
      // Token expirado o inválido -> Forzar cierre de sesión seguro
      authStore.logout();
    }
    return Promise.reject(error.response ? error.response.data : error);
  }
);

export default api;
```

---

## 🤖 6. Flujo de Integración del Chatbot Pasivo

Con el fin de mitigar riesgos de seguridad de API y evitar bloqueos por CORS o sobrecarga del lado del cliente, el componente interactivo flotante `ChatbotWidget.vue` delegará el procesamiento del lenguaje natural de manera pasiva a través de la API local.

### Esquema de Comunicación
1.  El usuario escribe un mensaje en la caja del widget flotante.
2.  El componente activa un estado local de "escribiendo..." visual (animación de tres puntos suspensivos rebotando) para dar feedback al usuario.
3.  Se despacha una llamada asíncrona no bloqueante a través de `chatbotService.js`:
    ```javascript
    // src/services/chatbotService.js
    import api from './api';

    export const sendMessageToChatbot = async (messageText) => {
      const response = await api.post('/chatbot', { message: messageText });
      return response.data; // Devuelve { reply: "Respuesta de IA limpia" }
    };
    ```
4.  La respuesta es renderizada dinámicamente en el histórico de la conversación con transiciones fluidas de scroll automático hacia la parte inferior del contenedor.

---

## 🛡️ 7. Seguridad y Rutas Protegidas (Navigation Guards)

Para proteger la integridad de los datos del usuario y asegurar que los administradores sean los únicos capaces de manipular el catálogo, la capa de enrutamiento en `src/router/index.js` gestionará de manera estricta los ciclos de navegación:

```javascript
import { createRouter, createWebHistory } from 'vue-router';
import { useAuthStore } from '@/store/auth';

const routes = [
  {
    path: '/',
    name: 'Home',
    component: () => import('@/views/HomeView.vue'),
  },
  {
    path: '/catalog',
    name: 'Catalog',
    component: () => import('@/views/CatalogView.vue'),
  },
  {
    path: '/product/:id',
    name: 'ProductDetail',
    component: () => import('@/views/ProductDetailView.vue'),
  },
  {
    path: '/cart',
    name: 'Cart',
    component: () => import('@/views/CartView.vue'),
  },
  {
    path: '/checkout',
    name: 'Checkout',
    component: () => import('@/views/CheckoutView.vue'),
    meta: { requiresAuth: true }, // Ruta Protegida
  },
  {
    path: '/login',
    name: 'Login',
    component: () => import('@/views/LoginView.vue'),
    meta: { guestOnly: true },
  },
  {
    path: '/register',
    name: 'Register',
    component: () => import('@/views/RegisterView.vue'),
    meta: { guestOnly: true },
  },
  {
    path: '/admin',
    name: 'AdminDashboard',
    component: () => import('@/views/AdminDashboardView.vue'),
    meta: { requiresAuth: true, requiresAdmin: true }, // Ruta de Administración Protegida
  },
];

const router = createRouter({
  history: createWebHistory(),
  routes,
  scrollBehavior() {
    return { top: 0 }; // Resetea el scroll de la ventana al navegar
  },
});

// Guardián Global de Navegación
router.beforeEach(async (to, from, next) => {
  const authStore = useAuthStore();
  
  // Si hay un token pero no tenemos datos del usuario, los recuperamos antes de decidir
  if (authStore.token && !authStore.user) {
    try {
      await authStore.fetchCurrentUser();
    } catch (e) {
      authStore.logout();
    }
  }

  const isAuthenticated = authStore.isAuthenticated;
  const isAdmin = authStore.isAdmin;

  if (to.meta.requiresAuth && !isAuthenticated) {
    // Redirige al login si requiere autenticación y no está logueado
    next({ name: 'Login', query: { redirect: to.fullPath } });
  } else if (to.meta.requiresAdmin && !isAdmin) {
    // Redirige al inicio si requiere rol de administrador y el usuario no lo tiene
    next({ name: 'Home' });
  } else if (to.meta.guestOnly && isAuthenticated) {
    // Redirige al Home si el usuario ya está autenticado e intenta ir a login/registro
    next({ name: 'Home' });
  } else {
    next();
  }
});

export default router;
```

---

## 🚀 8. Roadmap de Desarrollo del Frontend (6 Iteraciones Secuenciales)

De acuerdo al cronograma maestro, el desarrollo del cliente se estructurará de la siguiente manera:

### 📅 Iteración 1: Inicialización, Estructura y Átomos Base (Fase 1)
*   **Objetivos:** Configurar el andamiaje del proyecto y establecer el sistema de diseño visual.
*   **Tareas:**
    1.  Inicializar el proyecto frontend ejecutando `npx -y create-vite-app` con el template de Vue 3.
    2.  Instalar dependencias necesarias: `vue-router`, `pinia`, `axios`, `vitest` y `@vue/test-utils`.
    3.  Crear y estructurar el árbol completo de directorios en `src/`.
    4.  Implementar `variables.css` y `main.css` con el sistema HSL dinámico.
    5.  Codificar los componentes átomo reutilizables: `BaseButton.vue` (con loader integrado), `BaseInput.vue`, `BaseBadge.vue` y `BaseRating.vue`.

### 📅 Iteración 2: Enrutamiento, Autenticación y Perfiles (Fase 2)
*   **Objetivos:** Crear el sistema de navegación SPA seguro y la gestión de sesiones de usuario.
*   **Tareas:**
    1.  Configurar el router en `src/router/index.js` con soporte para lazy loading y transiciones de carga.
    2.  Codificar `useAuthStore` en Pinia para gestionar logins, registros, deslogueos y persistencia local del token JWT.
    3.  Desarrollar el servicio `authService.js` integrado con Axios.
    4.  Crear las vistas interactivas `LoginView.vue` y `RegisterView.vue` con validación de formularios en tiempo real.
    5.  Implementar el *Navigation Guard* global para controlar el acceso a zonas protegidas.

### 📅 Iteración 3: Catálogo, Búsqueda Avanzada y Detalle de Producto (Fase 3)
*   **Objetivos:** Presentar los artículos de forma interactiva y optimizada con capacidades de búsqueda reactiva.
*   **Tareas:**
    1.  Codificar `useProductStore` gestionando caché en cliente y estados de filtros.
    2.  Desarrollar `productService.js` con endpoints de consulta REST integrados de manera asíncrona.
    3.  Construir los componentes `SearchBar.vue`, `ProductCard.vue` y el organismo `ProductGrid.vue` con soporte para *skeleton loaders*.
    4.  Codificar `CatalogView.vue` organizando layouts responsivos mediante CSS Grid y Flexbox.
    5.  Crear `ProductDetailView.vue` mostrando ficha técnica, stock real, SKU y selectores dinámicos de cantidad.

### 📅 Iteración 4: Gestión de Carrito de Compras de Alta Precisión (Fase 4)
*   **Objetivos:** Implementar una cesta reactiva e impecable a nivel matemático que sincronice datos locales y remotos.
*   **Tareas:**
    1.  Codificar `useCartStore` implementando getters aritméticos de precisión financiera para Subtotal, IVA (21%) y Total Final.
    2.  Desarrollar `cartService.js` permitiendo el envío de ítems agregados de manera persistente a la base de datos tras loguearse.
    3.  Implementar animaciones tipo "pulse" de cambio de cantidad sobre `BaseBadge.vue` en `AppNavbar.vue`.
    4.  Codificar `CartView.vue` estructurando transiciones dinámicas al remover productos de la rejilla.

### 📅 Iteración 5: Pasarela de Pago Simulada y Transacciones (Fase 5)
*   **Objetivos:** Simular el flujo transaccional final en comunicación con el backend para la generación de pedidos.
*   **Tareas:**
    1.  Codificar el organismo `CheckoutForm.vue` con validación estricta de tarjetas bancarias simétricas y campos de envío.
    2.  Crear `CheckoutView.vue` detallando el resumen financiero final antes de comprometer la orden.
    3.  Desarrollar `orderService.js` consumiendo el endpoint `POST /api/orders` y controlando posibles excepciones (como stock insuficiente/overselling reportados por el backend).
    4.  Integrar vaciado automático del carrito en caso de transacción exitosa y redirección a una pantalla de felicitación/historial.

### 📅 Iteración 6: Soporte por Chatbot, QA y Certificación (Fase 6)
*   **Objetivos:** Implementar la asistencia virtual, ejecutar la suite de pruebas frontend de alto nivel y auditar la calidad del código.
*   **Tareas:**
    1.  Codificar `ChatbotWidget.vue` implementando scroll automático a final del scroll y feedback animado de "escribiendo...".
    2.  Integrar `chatbotService.js` enviando mensajes de manera pasiva mediante peticiones POST limpias al backend.
    3.  Escribir pruebas unitarias en **Vitest** con foco en la precisión decimal de los desgloses financieros en `cart.js`.
    4.  Comprobar el cumplimiento de contraste cromático y lectura semántica según la especificación WCAG 2.1 AA.
    5.  Someter el código a la auditoría estricta de `code-inspector` garantizando cero acoplamiento, alta cohesión y diseño SOLID.

## 📅 Iteración 7: Orquestación e Integración de la Vista Principal
*   **Objetivos:** Consolidar y orquestar de manera unificada todos los sistemas modulares independientes dentro del layout raíz de la página de inicio (HomeView), garantizando el flujo correcto de los almacenes de Pinia.
*   **Tareas:**
    1. Codificar la vista principal en `src/views/HomeView.vue` importando semánticamente los organismos construidos: `AppNavbar.vue`, `CategoryGrid.vue` y `ChatbotWidget.vue`.
    2. Vincular el ciclo de vida de la vista (`onMounted`) para disparar la inicialización del carrito del usuario (`initializeCart`) y verificar el estado de la sesión si existe un token persistido.
    3. Garantizar que no existan colisiones de estilos Vanilla CSS ni fugas de eventos reactivos en el contenedor principal (`<main class="home-container">`).
*   **Criterios de Aceptación:**
    *   **Cero Acoplamiento:** La vista HomeView.vue solo debe actuar como orquestadora estructural; no debe incluir llamadas directas a Axios ni mutaciones directas de estado fuera de las acciones de Pinia, manteniendo un diseño SOLID estricto evaluado con 10.0/10.0 por el inspector.
    *   **Estabilidad de Interfaz:** Al interactuar con el Chatbot o añadir elementos desde el grid, la barra de navegación debe reflejar reactivamente los cambios en tiempo real (como el contador de BaseBadge.vue) sin provocar re-renderizados completos de la página.
    *   **Accesibilidad Semántica Completa:** La composición total de la Home debe pasar la auditoría estructural de HTML5 (uso de etiquetas `<header>`, `<nav>`, `<main>`, `<footer>`) garantizando la navegación por teclado fluida y el enfoque visual de los componentes activos.

---

## 🏆 9. Criterios de Aceptación y QA (Definition of Done)

Para autorizar el merge final del frontend de NebriAmazon en la rama `main` y proceder con la liberación de la versión v1.0, el software frontend deberá acreditar lo siguiente:

1.  **Impecabilidad en los Cálculos:** La suite de pruebas de Vitest debe verificar que bajo cualquier combinación de precios unitarios y cantidades, los getters `subtotal`, `taxAmount` y `total` computen de forma exacta y no exhiban problemas de coma flotante de JavaScript.
2.  **Robustez de Estado de Red:** Toda petición asíncrona hacia el backend debe manejar de manera visual y clara los estados de carga (*loading loaders* bloqueando botones para evitar doble envío accidental) y renderizar mensajes de alerta comprensibles para el usuario en caso de error del servidor.
3.  **Cero Warnings en Consola:** La aplicación debe correr sin ningún tipo de error de renderizado, fugas de memoria por timers abiertos o warnings de reactividad en la consola de herramientas de desarrollo del navegador.
4.  **Cumplimiento de Accesibilidad:** Debe verificar exitosamente el contraste en pantalla de todas las vistas, y estructurar enlaces y botones con soporte completo para navegación por teclado (`tabindex` y `focus` visual).
5.  **Calidad Estructural:** El código resultante debe pasar la auditoría del `code-inspector`, certificando la inexistencia de lógica de negocio o de red directamente acoplada en las plantillas visuales de Vue, logrando una calificación perfecta de **10.0** en base a Clean Code y principios SOLID.
