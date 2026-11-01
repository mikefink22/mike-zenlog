# ADR-004: Stack Frontend Web Minimalista (React + Vite + Tailwind CSS + shadcn/ui + TanStack Query)

- **Estado**: Aceptado
- **Fecha**: 2026-11-01
- **Contexto**: La interfaz debe evitar la sobrecarga visual que hace que el usuario abandone su agenda física. Debe sentirse instantánea en escritorio y teléfono móvil (responsive), priorizando el flujo rápido de teclado y tacto de un Bullet Journal.

## Opciones Consideradas

1. **React + Vite + Tailwind CSS + shadcn/ui (Radix UI) + TanStack Query**.
2. **React + Material UI (MUI)**.
3. **Angular + Angular Material**.

## Decisión

Adoptar **React 19 + Vite + Tailwind CSS + shadcn/ui + TanStack Query**.

## Justificación

- **Estética Minimalista y Control Total**: `shadcn/ui` no es una librería cerrada en `node_modules`, sino componentes accesibles (basados en Radix UI) cuyo código vive en nuestro repositorio, permitiendo una estética limpia estilo Notion/Linear con soporte nativo para modo claro/oscuro.
- **Actualizaciones Optimistas (Optimistic UI)**: Con **TanStack Query**, cuando el usuario tilda una tarea o agrega un bullet en su Daily Log, la interfaz se actualiza en 0ms mientras sincroniza en segundo plano con la API.
- **Diseño Mobile-First Responsive**: Permite usar la aplicación desde el navegador del móvil en la Fase 2 (incluso instalable como PWA) antes de construir la app en React Native en la Fase 3.
