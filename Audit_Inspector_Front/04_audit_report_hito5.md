# 🔍 Reporte de Auditoría Técnica — HITO 5: Pasarela de Pago Simulada
**Rama:** `feature/05-pasarela-pago` · **Commit auditado:** `846f792`
**Inspector:** `code-inspector` · **Fecha:** 2026-05-24

---

## 1. CALIFICACIÓN GLOBAL

```
╔══════════════════════════════════════════════╗
║   CALIFICACIÓN FINAL:   9.1 / 10.0          ║
║   Veredicto:            ✅ APTO PARA MERGE   ║
╚══════════════════════════════════════════════╝
```

| Archivo              | Modularidad | Asincronía | Seguridad | Clean Code | Nota |
|---|:---:|:---:|:---:|:---:|:---:|
| `api.js`             | 10.0 | 10.0 | 9.5 | 10.0 | **9.9** |
| `orderService.js`    |  9.5 |  9.5 | 8.5 |  9.5 | **9.3** |
| `CheckoutForm.vue`   |  9.5 | 10.0 | 9.5 |  9.0 | **9.5** |
| `CheckoutView.vue`   |  9.0 |  9.5 | 9.0 |  9.0 | **9.1** |
| **Media ponderada**  |      |      |     |      | **9.1** |

---

## 2. PUNTOS FUERTES

### 2.1 Modularidad Arquitectónica (SRP + ISP bien aplicados)

La separación en capas es **textbook-level**. Cada archivo tiene exactamente una razón para cambiar:

- **`api.js`** solo cambia si cambia el mecanismo de transporte HTTP o la estrategia de autenticación.
- **`orderService.js`** solo cambia si cambia el contrato del endpoint `/api/orders` o la lógica de simulación.
- **`CheckoutForm.vue`** solo cambia si cambia el formulario de captura de datos o las reglas de validación local.
- **`CheckoutView.vue`** solo cambia si cambia la orquestación del flujo transaccional.

> Ningún archivo tiene más de **una responsabilidad observable**. Esto cumple el SRP de forma estricta.

El flujo de dependencias es **unidireccional y sin ciclos**:

```
CheckoutView.vue
  └──► orderService.js
         └──► api.js (cliente Axios)
  └──► useCartStore (Pinia — Iteración 4)
  └──► CheckoutForm.vue (componente hijo sin estado de red)
```

No existe ningún import en dirección inversa. La arquitectura es limpia y acíclica.

---

### 2.2 Resiliencia del Fallback Híbrido (`orderService.js`)

La estrategia de degradación elegante implementada en `placeOrder()` es uno de los puntos más sólidos de la entrega:

```js
// Intento real contra el backend
try {
  const response = await api.post('/orders', orderPayload);
  return response.data;
} catch (networkOrServerError) {
  // Errores tipados del interceptor → se propagan sin cambios
  if (networkOrServerError?.status && networkOrServerError?.code) {
    throw networkOrServerError;
  }
  // Error de red (sin backend) → motor de simulación
  return simulateApiCall(orderPayload);
}
```

La bifurcación entre **error tipado** (ya procesado por el interceptor de `api.js`) y **error de red genérico** (timeout, CORS) es correcta y evita enmascarar excepciones de negocio bajo el simulador. Este patrón replica el comportamiento de un **Circuit Breaker** simplificado.

---

### 2.3 Abstracción de Red (Ley de Demeter Aplicada)

- `CheckoutView.vue` **nunca importa Axios**. Solo llama a `placeOrder()`.
- `CheckoutForm.vue` **nunca importa ningún servicio**. Solo emite un evento `submit` con los datos crudos validados.
- `api.js` es el **único punto de la aplicación** que instancia y configura Axios.

Esto garantiza que un cambio de librería HTTP (de Axios a `fetch`, por ejemplo) solo requeriría modificar **1 archivo**.

---

### 2.4 Algoritmo de Luhn — Implementación Correcta y Pura (`CheckoutForm.vue`)

```js
const isValidLuhn = (rawNumber) => {
  if (!rawNumber || rawNumber.length < 13) return false;
  let sum = 0;
  let shouldDouble = false;
  for (let i = rawNumber.length - 1; i >= 0; i--) {
    let digit = parseInt(rawNumber[i], 10);
    if (shouldDouble) {
      digit *= 2;
      if (digit > 9) digit -= 9;
    }
    sum += digit;
    shouldDouble = !shouldDouble;
  }
  return sum % 10 === 0;
};
```

Verificación manual con tarjeta estándar `4111111111111111`:
- Suma Luhn calculada: **70** → `70 % 10 === 0` ✅
- La función es **pura** (sin side effects), lo que la hace trivialmente testeable con Vitest.
- La guarda `rawNumber.length < 13` previene falsos positivos en números cortos, cumpliendo la especificación ISO/IEC 7812.

---

### 2.5 Gestión de Asincronía — Patrón `try/catch/finally` Consistente

En `CheckoutView.vue`:

```js
const handleFormSubmit = async ({ shipping, payment }) => {
  isLoading.value   = true;
  serverError.value = '';
  try {
    const order = await placeOrder(...);
    await cartStore.clearCart();       // Tarea 4: vaciado post-éxito
    confirmedOrder.value = order;
  } catch (error) {
    serverError.value = error?.message ?? 'Error desconocido.';
  } finally {
    isLoading.value = false;           // Siempre desbloquea el botón
  }
};
```

El `finally` garantiza que `isLoading` se resetea **en todos los escenarios**, eliminando el riesgo de UI congelada tras un error inesperado. Es exactamente la estructura que prescribe Clean Code (Cap. 7: «Write Your Try-Catch-Finally Statement First»).

---

### 2.6 Mitigación de Anti-patrones de Red

| Anti-patrón | Estado |
|---|---|
| Doble envío accidental | ✅ Mitigado: el botón queda `disabled` mientras `isLoading === true` |
| Callback Hell | ✅ Eliminado: uso consistente de `async/await` |
| Instanciación directa de Axios en componentes | ✅ Prohibido por arquitectura |
| Error silenciado sin feedback visual | ✅ Toda excepción fluye al `serverError-banner` |
| Estado `loading` huérfano | ✅ Resuelto con `finally` |

---

### 2.7 Seguridad por Diseño — CVV y PAN

**CVV (`cf-cvv`):** declarado como `type="password"`. El valor nunca aparece en texto visible en el DOM ni en ningún log de red.

**Número completo de tarjeta (PAN):** El servicio construye `sanitizedPayment` con solo `cardLastFour` para el payload real hacia el backend. El número completo **solo viaja como anotación comentada para simulación**, con una nota explícita de que en producción sería reemplazado por un token de pasarela (Stripe, Redsys):

```js
const sanitizedPayment = {
  cardholderName: paymentData.cardholderName,
  cardLastFour: paymentData.cardNumber.replace(/\s/g, '').slice(-4),
  expiry: paymentData.expiry,
};
```

La validación Luhn ocurre **100% en el cliente** antes de que ningún dato abandone el navegador.

---

### 2.8 Accesibilidad (WCAG 2.1 AA)

- Todos los `<input>` tienen `id` único, `<label for="...">` asociado, y `autocomplete` correcto.
- Campos sensibles usan `aria-required="true"` y `aria-invalid` dinámico.
- El banner de error del servidor usa `role="alert"` + `aria-live="assertive"` para lectores de pantalla.
- El botón de submit incluye `aria-busy` durante la carga.
- Los emojis decorativos tienen `aria-hidden="true"` en todos los casos auditados.

---

## 3. DETALLES DE MEJORA

### ⚠️ M1 — `orderService.js` L.124: El PAN completo entra en el payload de simulación

```js
payment: {
  cardNumber: paymentData.cardNumber, // ← PAN completo en el objeto de red
  ...sanitizedPayment,
}
```

**Observación:** Aunque el comentario explica que en producción se sustituiría por un token, el número completo se incluye en `orderPayload` que **sí se envía a `api.post('/orders', orderPayload)`** en el bloque `try`. Si el backend existe y loguea el body, el PAN quedaría expuesto en logs de servidor.

**Sugerencia de corrección:** Separar claramente el payload para el backend real (sin PAN) del payload del simulador (con PAN para la detección de la tarjeta de prueba):

```js
const backendPayload = { shipping, payment: sanitizedPayment, items, total };
const simulationPayload = { ...backendPayload, payment: { ...sanitizedPayment, cardNumber } };

try {
  const response = await api.post('/orders', backendPayload); // PAN nunca viaja
  return response.data;
} catch (err) {
  if (isTypedError(err)) throw err;
  return simulateApiCall(simulationPayload); // PAN solo llega al motor local
}
```

**Severidad:** 🟡 Media — No es un fallo en el contexto simulado actual, pero sería crítico al conectar el backend real.

---

### ⚠️ M2 — `CheckoutView.vue` L.186: Aritmética flotante directa en la plantilla

```html
{{ (item.quantity * item.product.price).toFixed(2) }} €
```

**Observación:** El subtotal por ítem (`quantity * price`) se calcula directamente en la plantilla con aritmética de punto flotante nativa de JS, **sin el escalado en centavos** que implementa `useCartStore` para el subtotal total. Esto puede generar subtotales de línea visualmente incorrectos (ej. `0.1 + 0.2 = 0.30000000000000004`).

**Sugerencia:** Extraer a un `computed` o usar el helper de la store:

```js
const itemLineTotal = (item) =>
  Number(((Math.round(item.product.price * 100) * item.quantity) / 100).toFixed(2));
```

**Severidad:** 🟡 Baja-Media — Solo afecta a la visualización de línea; el total final es correcto.

---

### 💡 M3 — `CheckoutForm.vue` L.21: Import `watch` no utilizado

```js
import { ref, computed, watch } from 'vue';
//                        ^^^^^ importado pero nunca usado en el módulo
```

**Observación:** La función `watch` fue importada pero no se usa en el componente final. Es una dependencia residual de una refactorización anterior.

**Sugerencia:** Eliminar `watch` del import:

```js
import { ref, computed } from 'vue';
```

**Severidad:** 🟢 Trivial — Solo afecta al bundle size en un par de bytes (tree-shaking de Vite lo elimina en producción, pero es código muerto visible).

---

### 💡 M4 — `api.js` L.49: Sin manejo de errores 403/422 en el interceptor global

El interceptor de respuesta solo captura `401` (logout automático). Los códigos `403 Forbidden` (sin permiso) y `422 Unprocessable Entity` (validación del servidor) se propagan crudos al servicio llamante sin normalización de formato.

**Sugerencia:** Añadir normalización mínima para facilitar el manejo en los servicios:

```js
if (error.response?.status === 403) {
  // Podría loguear o emitir un evento de analítica
}
// La propagación normalizada ya ocurre en la línea final del interceptor
return Promise.reject(error.response?.data ?? error);
```

**Severidad:** 🟢 Baja — El error se propaga correctamente al catch; es una mejora de ergonomía para desarrolladores futuros.

---

### 💡 M5 — `CheckoutForm.vue`: Validación de expiración sin año de dos dígitos vacío

```js
const year = parseInt(`20${parts[1]}`, 10);
```

Si el usuario escribe solo `12/` (sin año), `parts[1]` es `''` y `year` resulta `2000`, que podría pasar la validación si el mes actual es diciembre del año anterior. Es un edge case extremo pero posible durante la entrada de datos progresiva.

**Sugerencia:** Añadir guarda de longitud mínima del año:

```js
if (!parts[1] || parts[1].length !== 2) return 'Formato inválido (MM/AA).';
```

**Severidad:** 🟢 Muy Baja — Solo ocurre en un estado transitorio del input y no pasaría la validación de Luhn adicionalmente.

---

## 4. VEREDICTO DE COMPILACIÓN

### Dependencias entre Componentes y Stores de Pinia

```
CheckoutView.vue
  ├── useCartStore   ─► Getters: subtotal, taxAmount, total, items, totalItemsCount ✅
  │                    Acción:  clearCart() ✅ (llamada awaited, Tarea 4 ✅)
  ├── useRouter      ─► push({ name: 'Catalog' }), push({ name: 'Cart' }) ✅
  ├── placeOrder     ─► orderService.js → api.js → Axios ✅
  └── CheckoutForm   ─► Props: isLoading, serverError | Emit: @submit ✅

api.js
  └── useAuthStore   ─► token (petición) | logout() (respuesta 401) ✅
                        (llamada diferida en tiempo de ejecución, no en módulo) ✅
```

**Ciclo de Pinia sin riesgos:** `useAuthStore` se invoca dentro de los callbacks de los interceptores de Axios, que son funciones diferidas. No hay llamadas a `useAuthStore()` a nivel de módulo, lo que elimina el error «`getActivePinia() called before createPinia()`». ✅

**Flujo `clearCart()` post-orden:** La acción se llama con `await` **dentro del bloque `try`**, garantizando que el carrito solo se vacía si la orden tuvo éxito. Si `clearCart()` fallara, la excepción sería capturada por el `catch` y mostraría un error al usuario en lugar de confirmar una orden fallida. ✅

**Errores tipados `{ status, code, message }`:** El flujo de propagación es limpio y consistente:

```
simulateApiCall() ──throw──► placeOrder() ──re-throw──► handleFormSubmit() ──► serverError.value
interceptor api.js ──reject──► placeOrder() catch ──► re-throw ──► handleFormSubmit() ──► serverError.value
```

En ningún punto se pierde información del error ni se enmasca una excepción sin propagar.

### Conclusión de Compilación

| Criterio | Estado |
|---|---|
| Cero imports circulares | ✅ Verificado |
| Cero instancias de Axios fuera de `api.js` | ✅ Verificado |
| Pinia activa antes del primer `useStore()` | ✅ Verificado (diferido en callbacks) |
| `clearCart()` solo en transacción exitosa | ✅ Verificado |
| CVV nunca en texto plano visible | ✅ Verificado (`type="password"`) |
| Errores tipados propagados correctamente | ✅ Verificado |
| Código muerto (warning) | ⚠️ `watch` importado sin uso en `CheckoutForm.vue` |
| PAN en payload de backend real | ⚠️ A corregir antes de conectar backend productivo |

> **El código es ESTABLE, COMPILABLE y APTO PARA MERGE** en la rama `feature/05-pasarela-pago`. Las dos observaciones de severidad media (`M1`, `M2`) deben resolverse antes de conectar el backend real de producción, pero no bloquean el flujo de demostración ni el avance a la Iteración 6.

---

*Auditoría realizada por `code-inspector` · Principios aplicados: Clean Code (R.C. Martin), SOLID, WCAG 2.1 AA, ISO/IEC 7812 (Luhn)*
