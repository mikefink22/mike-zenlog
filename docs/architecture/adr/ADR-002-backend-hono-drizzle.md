# ADR-002: Framework Backend con Hono y Drizzle ORM

- **Estado**: Aceptado
- **Fecha**: 2026-11-01
- **Contexto**: El backend debe poder correr localmente como un servidor HTTP tradicional en Node.js durante la Fase 1 y desplegarse sin fricción en funciones serverless (**AWS Lambda**) en la Fase 2, sirviendo tanto a la aplicación Web como a la futura App Mobile.

## Opciones Consideradas

1. **Hono + Drizzle ORM**: Framework web basado en Web Standards (`Request`/`Response`) con adaptadores oficiales para Node.js y AWS Lambda, combinado con Drizzle ORM (SQL-first, liviano, sin binarios pesados en Rust).
2. **NestJS + Prisma ORM**: Framework empresarial con decoradores e inyección de dependencias; Prisma usa un motor de consultas que incrementa el peso del bundle y el *cold start* en AWS Lambda.
3. **Express + Sequelize/TypeORM**: Stack tradicional de Node.js, con soporte deficiente para inferencia de tipos moderna en TypeScript y necesidad de wrappers adicionales para serverless.

## Decisión

Adoptar **Hono** como framework HTTP y **Drizzle ORM** para el acceso a datos SQL.

## Justificación

- **Compatibilidad Dual (Local + AWS Lambda)**: Hono usa el mismo código de rutas tanto en `@hono/node-server` (local) como en `hono/aws-lambda` (producción), sin reescribir controladores.
- **Cero Cold-Start Overhead**: Tanto Hono como Drizzle tienen un tamaño de bundle mínimo (~15-30 KB), ideal para la capa gratuita de AWS Lambda donde la memoria y el tiempo de arranque importan.
- **Tipado End-to-End (Hono RPC + Zod)**: Permite validar requests con `@hono/zod-validator` reutilizando directamente los esquemas de `packages/shared`.

## Consecuencias

- **Positivas**: Altísimo rendimiento, consultas SQL transparentes y tipadas, migraciones SQL versionables con `drizzle-kit`.
- **Negativas / Mitigación**: Menos "magia" automática que Django REST Framework, por lo que estructuraremos `apps/api` en capas claras (`routes`, `services`, `repositories/db`, `middlewares`).
