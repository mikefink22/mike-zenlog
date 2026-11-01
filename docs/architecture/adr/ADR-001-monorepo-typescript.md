# ADR-001: Arquitectura de Monorepo Full-Stack en TypeScript con `npm workspaces`

- **Estado**: Aceptado
- **Fecha**: 2026-11-01
- **Contexto**: Se requiere desarrollar una Agenda minimalista estilo Bullet Journal que comience ejecutándose localmente, luego se despliegue en la web de forma responsive y en una tercera etapa sume una aplicación móvil nativa/multiplataforma (iOS/Android), compartiendo un mismo Backend.

## Opciones Consideradas

1. **Monorepo Full-Stack TypeScript (`npm workspaces`)**:
   - `packages/shared` (Esquemas Zod y tipos compartidos)
   - `apps/api` (Backend en Node.js + TypeScript)
   - `apps/web` (Frontend en React + Vite + TypeScript)
   - `apps/mobile` (Futuro React Native / Expo en TypeScript)
2. **Polirepo con lenguajes heterogéneos (Python/DRF en Backend + React en Web + Java/Kotlin en Mobile)**.
3. **Backend en Java (Spring Boot) + Frontend en Angular/React**.

## Decisión

Adoptar la **Opción 1: Monorepo Full-Stack en TypeScript gestionado con `npm workspaces` (v11.14.1)**.

## Justificación

- **Fuente única de verdad (`packages/shared`)**: Definir los contratos de datos y validaciones con **Zod** una sola vez evita desincronización entre lo que espera la API, lo que envía la Web y lo que consumirá la App Mobile.
- **Seguridad de entorno**: `npm` v11 ya está instalado y firmado junto con Node.js v24 en la estación de trabajo, soportando `workspaces` nativamente sin requerir binarios externos adicionales.
- **Transferencia de conocimiento hacia Mobile**: El uso de React + TypeScript en `apps/web` reduce drásticamente la curva de aprendizaje para `apps/mobile` con React Native (Expo) en la Fase 3.

## Consecuencias

- **Positivas**: Refactorizaciones seguras de punta a punta; un solo comando `npm install` y `npm test` en la raíz; tipado compartido sin publicar paquetes privados a npm.
- **Negativas / Mitigación**: Requiere disciplina para no acoplar código de UI dentro de `packages/shared` (este paquete solo debe contener esquemas Zod, tipos puros y utilidades de dominio agnósticas de plataforma).
