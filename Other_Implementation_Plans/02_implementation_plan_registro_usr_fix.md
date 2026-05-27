# Plan de Implementación: Flujos y Vistas de Autenticación (Login & Registro)

Este plan detalla los pasos para sustituir las páginas de prueba actuales de la autenticación por los componentes reales y funcionales del sistema en `frontend-nebri-amazon`, siguiendo los principios de **Clean Code** y garantizando una experiencia visual premium y adaptada al sistema de diseño existente.

## User Review Required

> [!IMPORTANT]
> - **Ubicación de Vistas:** Guardaremos las vistas `LoginView.vue` y `RegisterView.vue` directamente dentro de la carpeta `src/views/` (donde ya existe el cascarón de `LoginView.vue`), garantizando consistencia con la estructura de archivos plana de la aplicación.
> - **Componente Base Reutilizable:** Extraeremos un componente base `BaseInput.vue` en `src/components/atoms/` para evitar duplicación de estilos de formulario, clases de error y atributos de accesibilidad (`aria-invalid`, etc.).
> - **Actualización Dinámica de AppNavbar:** Refactorizaremos el componente `AppNavbar.vue` para que use el `authStore` de Pinia de manera reactiva, mostrando `"Hola, [Nombre]"` y un enlace para cerrar sesión cuando el usuario esté autenticado.

## proposed-changes

---

### [Componente Átomo Reutilizable]

Crearemos un componente `BaseInput` para encapsular la estructura y estilos CSS de los campos de entrada de texto, reduciendo de forma drástica la duplicación de código en los formularios de la app.

#### [NEW] [BaseInput.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/components/atoms/BaseInput.vue)
- Campo de entrada reactivo (`v-model` compatible con Vue 3).
- Soporte para etiquetas personalizadas, campos obligatorios (asterisco rojo sutil), placeholders y tipos (texto, email, password).
- Renderizado de errores elegantes justo debajo del input con animación suave (`@keyframes errorSlideIn`).
- Accesibilidad incorporada (`aria-invalid`, `aria-required` y `aria-describedby` dinámicos).

---

### [Cabecera Global]

Haremos la barra de navegación inteligente, adaptando su contenido visual según el estado de autenticación del usuario.

#### [MODIFY] [AppNavbar.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/components/organisms/AppNavbar.vue)
- Importar y conectar el `useAuthStore` de Pinia de manera reactiva.
- Si el usuario no está autenticado, mantener el botón original de `"Hola, Identifícate"`.
- Si el usuario está autenticado, cambiar el texto dinámicamente a `"Hola, [Nombre]"` y añadir funcionalidad directa para `"Cerrar Sesión"` mediante la acción `authStore.logout()`.

---

### [Vistas de Autenticación]

Implementaremos las vistas completas de login y registro. Ambas mantendrán el cascarón de diseño (Header, Footer, Chatbot) para ser páginas independientes consistentes.

#### [MODIFY] [LoginView.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/views/LoginView.vue)
- Layout centrado y estético usando el sistema de colores premium del proyecto (`variables.css`).
- Tarjeta de autenticación con campos para **Email** y **Contraseña** usando el nuevo componente `BaseInput`.
- Botón de envío con indicador de carga (`loading` y spinner animado) interactuando con `authStore.login()`.
- Validaciones en cliente reactivas antes de enviar (campos obligatorios y formato de email).
- Enlace "¿No tienes cuenta? Regístrate aquí" que redirige dinámicamente a la ruta de registro.
- Banner de error global en caso de fallo en la autenticación por servidor.

#### [NEW] [RegisterView.vue](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/views/RegisterView.vue)
- Tarjeta con la misma consistencia visual y estética.
- Campos para **Nombre completo**, **Email**, **Contraseña** y **Confirmar contraseña** usando `BaseInput`.
- Botón de envío con indicador de carga (`loading` y spinner animado) interactuando con `authStore.register()`.
- Validaciones completas antes de enviar:
  - Campos requeridos.
  - Formato de email válido.
  - Contraseña con longitud mínima de 6 caracteres (restricción del backend mockeado).
  - Coincidencia exacta entre "Contraseña" y "Confirmar contraseña".
- Redirección automática y login fluido tras un registro exitoso.
- Enlace "¿Ya tienes una cuenta? Inicia sesión aquí" hacia la ruta de login.

---

### [Enrutamiento]

#### [MODIFY] [index.js](file:///d:/Inst_Nebrija_2025/CLASES_IABD_%28G1%29/Programaci%C3%B3n_de_IA/Prof_Rub%C3%A9n/Proyectos/ProyectoNebriAmazon/frontend-nebri-amazon/src/router/index.js)
- Importar estáticamente o por carga diferida (`lazy load`) la nueva vista `RegisterView.vue`.
- Declarar la ruta `/register` asignándole el nombre `'Register'`.

## Verification Plan

### Automated / Manual Verification
1. **Verificación de Navegación:**
   - Clicar en "Hola, Identifícate" en el Navbar de Home para navegar a `/login`.
   - Clicar en "¿No tienes cuenta? Regístrate aquí" para navegar a `/register`.
   - Clicar en "¿Ya tienes una cuenta? Inicia sesión aquí" para volver a `/login`.
   - Clicar en el logo "NebriAmazon" en cualquiera de las vistas para volver a la Home `/`.
2. **Validaciones de Formulario:**
   - Intentar enviar el formulario de Login vacío y verificar que aparecen mensajes de error rojos y elegantes en tiempo real.
   - Introducir un email inválido (ej: `test`) y comprobar la alerta de formato.
   - Intentar registrarse con contraseñas que no coinciden y comprobar el mensaje correspondiente.
3. **Flujo Completo de Autenticación:**
   - Realizar un Login con credenciales simuladas. Comprobar que el botón cambia a "Procesando..." y el formulario se bloquea transitoriamente (800ms de latencia).
   - Verificar la redirección exitosa a la página de Inicio.
   - Confirmar que el Navbar actualiza reactivamente su contenido a `"Hola, [Nombre]"` y muestra `"Cerrar Sesión"`.
   - Clicar en `"Cerrar Sesión"` en el Navbar y verificar que el usuario se desautentica de forma segura, volviendo al estado inicial.
