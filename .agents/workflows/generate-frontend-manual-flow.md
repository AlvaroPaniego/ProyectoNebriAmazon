---
description: Multi-agent pipeline to audit the frontend workspace and automatically generate the official architecture and WCAG 2.1 AA accessibility manual in Spanish.
---

[SYSTEM INSTRUCTION: Execute the installed skill 'documentation-writer' by coordinating the local profiles defined in the .agents/ directory].

# 🚀 Pipeline: generate-frontend-manual-flow

Sequentially automate the engineering documentation pipeline over the current workspace:

1. **Analysis Phase:** Invoke the instructions from `.agents/frontend-code-reader.md` to analyze the atomic structure, components, and Pinia state management inside `frontend-nebri-amazon/src/`.
2. **Transfer Phase:** Pass the generated structural payload directly into the execution context of the agent `.agents/frontend-technical-writer.md`.
3. **Generation Phase:** Synthesize the technical insights and write the official system manual in the local path: `Technical-Docs/01_manual_tecnico_frontend.md`.

## 📋 Deliverable Requirements (CRITICAL: Output must be written in SPANISH)
The generated manual must be structured in Spanish using premium Markdown formatting with the following sections:
- # 📘 Manual de Arquitectura y Componentes Frontend - NebriAmazon
- ## 🏗️ 1. Estructura y Enrutamiento (Vistas, Router y layouts).
- ## 🍍 2. Gestión de Estado Global (Análisis del flujo de datos dinámico en Pinia).
- ## 🎨 3. Sistema de Diseño Atómico y Tokens (Variables HSL globales estilo Amazon).
- ## ♿ 4. Cumplimiento de Estándares WCAG 2.1 AA (Matriz de accesibilidad, aria-labels y foco).