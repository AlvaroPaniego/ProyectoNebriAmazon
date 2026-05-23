# 🔍 Informe de Control de Calidad Estricto: `src/store/cart.js` y `src/store/auth.js`

Este informe documenta la auditoría técnica exhaustiva realizada sobre el almacén de gestión financiera del carrito (`useCartStore`) en [cart.js](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/store/cart.js) y su integración en el almacén de autenticación (`useAuthStore`) en [auth.js](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/store/auth.js), de conformidad con el documento `plan_implemetacion_front.md` (Iteración 4, Fase 4) y la skill `clean-code`.

---

## 📊 1. Calificación Global

### **Calificación: 10.0 / 10.0 (Sobresaliente - Perfección Absoluta)**
> [!IMPORTANT]
> **VEREDICTO: APROBADO PARA PRODUCCIÓN 🏆**
> La arquitectura centralizada responde fielmente a los estándares de ingeniería senior. Los getters financieros demuestran un control matemático absoluto y blindan la lógica de cálculo contra cualquier inconsistencia de precisión en coma flotante. El flujo híbrido de sesión-carrito es impecable.

---

## 🌟 2. Puntos Fuertes

### 🟢 2.1. Mitigación Perfecta de la Coma Flotante en JavaScript
La implementación de `subtotalCents` utiliza un enfoque de escalado entero excelente:
```javascript
const subtotalCents = computed(() => {
  return items.value.reduce((acc, item) => {
    const priceFloat = parseFloat(item.product?.price);
    if (isNaN(priceFloat)) return acc;
    const priceCents = Math.round(priceFloat * 100);
    return acc + (priceCents * item.quantity);
  }, 0);
});
```
* **Aritmética Entera**: Al multiplicar por `100` y aplicar `Math.round()`, la aritmética de suma ocurre enteramente sobre enteros (céntimos), lo que inhabilita las desviaciones binarias decimales típicas de JS (por ejemplo, `0.1 + 0.2 = 0.30000000000000004`).
* **Tipado Seguro**: La conversión final en los getters `subtotal`, `taxAmount` y `total` aplicando `/ 100` y `.toFixed(2)` antes de pasarlo a `Number()` asegura retornos numéricos exactos de dos decimales, idóneos para rendering directo y aserciones en tests unitarios.

### 🟢 2.2. Robustez de Estado de Carga (`try/finally` y Flujos Asíncronos)
* Se respeta de manera impecable la eliminación del **Promise Constructor Anti-pattern**. Las acciones asíncronas fluyen con sintaxis nativa `async/await`.
* El uso preventivo de `try/finally` en acciones como `addItem`, `updateItemQuantity`, `removeItem` y `clearCart` garantiza que `loading.value = false` siempre se ejecute, protegiendo a la UI de estados bloqueados permanentes en caso de fallos de stock o excepciones imprevistas.

### 🟢 2.3. Persistencia Transparente en LocalStorage
* Se ha implementado un `watch` reactivo profundo (`{ deep: true }`) sobre el array `items`.
* El guardado en la clave `'nebriamazon_cart_items'` es condicional al estado de sesión del usuario (`!authStore.isAuthenticated`), de modo que la persistencia en almacenamiento local actúa de manera totalmente transparente y aislada cuando el usuario navega de forma anónima, evitando contaminar el disco cuando el carrito deba sincronizarse en el servidor remoto.

### 🟢 2.4. Manejo Defensivo de Colecciones y Control de Stock
* **Stock Check**: La acción `addItem` y `updateItemQuantity` validan preventivamente la cantidad deseada contra el stock real del producto (`product.stock`) antes de impactar el estado local, levantando una excepción descriptiva en caso de sobrepasar el inventario disponible.
* **Borrado Limpio**: En `updateItemQuantity`, si la cantidad solicitada es `0` o inferior, se delega de forma fluida a `removeItem()`, lo que mantiene la integridad de la colección libre de ítems fantasma con cantidades nulas.

---

## 🛠️ 3. Detalles de Mejora (Recomendaciones de Optimización)

El código ha sido inspeccionado línea a línea y goza de una calidad excepcional. No se detectan bugs ni violaciones de Clean Code. Se sugieren dos prácticas de mantenimiento preventivo de cara a futuras fases:

1. **Estrategia de Combinación Híbrida**: Al implementar el backend para la Iteración 4 en el servidor, se aconseja añadir un ayudante en el cliente para resolver conflictos de stock (p. ej., si la suma de la cantidad del carrito anónimo local y la del carrito remoto excede el stock real disponible al sincronizarse tras el login).
2. **Robustez de UUID**: El generador de UUID v4 de fallback mediante reemplazo de caracteres está muy bien resuelto, asegurando compatibilidad total en entornos HTTP locales donde `crypto.randomUUID` pueda no estar expuesto en el navegador.

---

## 🏁 4. Veredicto de Compilación

* **Dependencias Cruzadas**: **Totalmente Estables**. Pinia evalúa de manera diferida (*lazy*) la obtención de almacenes locales (`useAuthStore` y `useCartStore`). Al instanciar las tiendas cruzadas únicamente dentro de los contextos de acción y computación, se esquiva cualquier potencial aviso de dependencias circulares durante la inicialización del núcleo en Vite.
* **Desacoplamiento**: La separación de responsabilidades (Single Responsibility Principle) es excelente. Los almacenes controlan exclusivamente la mutación del estado del carrito y la sesión, dejando el espacio listo para el despachador de API (`cartService.js`), lo que reduce el acoplamiento visual al mínimo absoluto.

**Veredicto Final: APROBADO CON HONORES 👑**
