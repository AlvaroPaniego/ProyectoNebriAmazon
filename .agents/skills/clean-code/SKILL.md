---
name: clean-code
description: 'Esta skill encapsula los principios de "Clean Code" de Robert C. Martin y la arquitectura SOLID para auditar y transformar código funcional en software impecable, legible y altamente mantenible.'
risk: safe
source: 'NebriAmazon Quality Core'
date_added: '2026-05-20'
---

# 💎 Skill de Calidad de Código: Clean Code & SOLID

Esta skill define las directrices y estándares de excelencia para auditar, refactorizar y verificar código fuente. Su objetivo es asegurar que cada módulo, clase o función sea fácil de leer, entender y extender por cualquier desarrollador.

---

## 🧠 Filosofía Core del Código Limpio

> "Cualquier tonto puede escribir código que una computadora entienda. Los buenos programadores escriben código que los humanos pueden entender." — Martin Fowler

---

## 1. 📛 Nombres Significativos y Descriptivos

Los nombres en el código deben revelar completamente la intención:

*   **Revelación de Intención:** `elapsedTimeInDays` en lugar de `d`. `activeUsers` en lugar de `list`.
*   **Evitar la Desinformación:** No llamar `accountList` a una variable que es de tipo `Set` o `Map`.
*   **Distinciones Significativas:** Evitar nombres de relleno como `ProductData`, `ProductInfo` o `ProductObject` para clases distintas en el mismo contexto.
*   **Nombres de Clases:** Deben ser sustantivos o frases nominales (`User`, `ShoppingCart`, `InvoiceParser`). Evitar sufijos genéricos como `Manager`, `Data` o `Helper`.
*   **Nombres de Métodos:** Deben ser verbos o frases verbales (`saveOrder`, `calculateTotal`, `isEmailValid`).

---

## 🧱 2. Funciones y Métodos Limpios

Las funciones deben ser estructuradas bajo la regla de mínima complejidad:

*   **¡Pequeñas!:** Las funciones deben ser lo más cortas posible (generalmente menos de 20 líneas).
*   **Hacer una Sola Cosa (Single Responsibility):** Una función debe hacer exactamente una cosa, hacerla bien y ser la única responsable de esa lógica.
*   **Un Solo Nivel de Abstracción:** No mezclar lógica de negocio de alto nivel con detalles técnicos de bajo nivel (como expresiones regulares o formateo de strings) dentro de la misma función.
*   **Parámetros / Argumentos:** El número ideal de argumentos es 0. 1 o 2 es aceptable. 3 requiere una justificación sólida. Evitar a toda costa funciones con más de 3 argumentos (utilizar objetos de configuración o DTOs en su lugar).
*   **Sin Efectos Secundarios:** Una función no debe modificar el estado global de forma inesperada ni alterar parámetros mutables a menos que sea su propósito explícito.

---

## 💬 3. Comentarios Justificados

Los comentarios deben servir para explicar la *intención*, no para encubrir código confuso:

*   **No comentes código malo, refactorízalo:** La necesidad de un comentario explicativo suele ser un síntoma de que el código no es lo suficientemente expresivo.
*   **Comentarios Buenos:** Avisos legales, explicaciones de algoritmos matemáticos complejos o expresiones regulares no obvias, advertencias sobre consecuencias secundarias, TODOs estructurados.
*   **Comentarios Malos:** Comentarios redundantes que repiten lo que hace el código línea por línea, comentarios desactualizados (desinformación), código comentado (usar control de versiones en su lugar).

---

## 📐 4. Principios de Diseño SOLID

La base del diseño orientado a objetos mantenible y modular:

1.  **Single Responsibility Principle (SRP) / Única Responsabilidad:** Una clase debe tener una, y solo una, razón para cambiar.
2.  **Open/Closed Principle (OCP) / Abierto/Cerrado:** Las entidades de software deben estar abiertas para su extensión, pero cerradas para su modificación.
3.  **Liskov Substitution Principle (LSP) / Sustitución de Liskov:** Las subclases o tipos derivados deben poder sustituir por completo a sus clases base sin alterar el comportamiento correcto del programa.
4.  **Interface Segregation Principle (ISP) / Segregación de Interfaces:** Es mejor tener muchas interfaces específicas que una sola interfaz de propósito general. Ningún cliente debe ser obligado a depender de métodos que no utiliza.
5.  **Dependency Inversion Principle (DIP) / Inversión de Dependencias:** Depender de abstracciones (interfaces o clases abstractas), no de concreciones (clases específicas). Inyectar dependencias en lugar de instanciarlas manualmente dentro del objeto.

---

## 🧪 5. Tests Unitarios Limpios

Las pruebas automatizadas deben ser tratadas con el mismo estándar de calidad que el código de producción:

*   **Los Tres Pasos de TDD (Test-Driven Development):**
    1.  Escribir una prueba que falle.
    2.  Escribir el código mínimo para que la prueba pase.
    3.  Refactorizar el código garantizando que la prueba siga pasando.
*   **Principios F.I.R.S.T. para Tests:**
    *   **F**ast (Rápidos): Los tests deben ejecutarse instantáneamente.
    *   **I**ndependent (Independientes): Un test no debe depender del resultado de otro.
    *   **R**epeatable (Repetibles): Deben dar el mismo resultado en cualquier entorno.
    *   **S**elf-Validating (Auto-validados): Tienen un veredicto claro de éxito o fallo (Boolean).
    *   **T**imely (Oportunos): Se escriben antes o en paralelo al código de producción.

---

## 🛠️ Checklist de Auditoría del Inspector

- [ ] ¿El código está completamente libre de variables o métodos con nombres crípticos (ej. `x`, `temp`, `data`)?
- [ ] ¿Cada función o clase realiza exactamente una única tarea (SRP)?
- [ ] ¿Se inyectan las dependencias externas en lugar de instanciarse con `new` o equivalentes dentro del código?
- [ ] ¿El código está libre de bloques duplicados (principio DRY - Don't Repeat Yourself)?
- [ ] ¿El manejo de excepciones captura errores específicos y amigables en lugar de propagar excepciones genéricas?
- [ ] ¿Se cuenta con una suite de tests unitarios robusta que verifique caminos felices y límites de error?
