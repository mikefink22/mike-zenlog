# Tablero Kanban del Proyecto — Agenda 2026

> **Metodología**: GitFlow + BDD (Gherkin Acceptance Criteria).
> Cada tarjeta representa un Issue trazable a [docs/backlog/USER_STORIES.md](./backlog/USER_STORIES.md).
> Ninguna tarjeta pasa a `Done` sin tests automatizados en verde y actualización del [Cuaderno de Desarrollo](./learning/DEV_NOTEBOOK.md).

---

## 📊 Estado Actual por Columnas

| Columna | Cantidad | Descripción |
| :--- | :---: | :--- |
| **Done** | 1 | Estructura de ingeniería, ADRs, SRS, GitFlow, Hooks de seguridad y Kanban inicial |
| **In Review / Testing** | 0 | Esperando validación de tests y merge de PR hacia `develop` |
| **In Progress** | 0 | En desarrollo activo en una rama `feature/ISSUE-XXX-*` |
| **Ready (Sprint Fase 1 - MVP Local)** | 3 | Listas para comenzar en orden de dependencia |
| **Backlog (Fase 1 UI, Fase 2 AWS, Fase 3 Mobile)** | 9 | Planificadas y especificadas con criterios de aceptación |

---

## 🟢 1. Ready (Próximas a tomar desde `develop` — Fase 1 Core)

- [ ] **[ISSUE-001](./backlog/USER_STORIES.md#issue-001-configuración-del-monorepo-typescript-y-esquemas-zod-compartidos-packagesshared)**: Configuración del Monorepo TypeScript y Esquemas Zod Compartidos (`packages/shared`)
  - **Rama**: `feature/ISSUE-001-shared-zod-schemas`
  - **Prioridad**: Alta (Bloqueante para API y Web)
  - **Entregables**: Esquemas Zod (`BulletItem`, `CreateBullet`, `UpdateBullet`, extractor de `#tags`) + Suite Vitest.
- [ ] **[ISSUE-002](./backlog/USER_STORIES.md#issue-002-capa-de-base-de-datos-con-drizzle-orm-y-pglite-appsapi)**: Capa de Base de Datos con Drizzle ORM y PGlite (`apps/api`)
  - **Rama**: `feature/ISSUE-002-drizzle-pglite-db`
  - **Prioridad**: Alta
  - **Entregables**: Tablas SQL `users` y `bullets`, conexión dual PGlite/Postgres + Tests con PGlite en memoria.
- [ ] **[ISSUE-003](./backlog/USER_STORIES.md#issue-003-autenticación-dual-modo-local-single-user--login-jwt-appsapi)**: Autenticación Dual (Modo Local Single-User + Login JWT) (`apps/api`)
  - **Rama**: `feature/ISSUE-003-auth-local-and-jwt`
  - **Prioridad**: Alta
  - **Entregables**: Middleware Auth en Hono, endpoints `/api/auth/login` y `/api/auth/me` + Tests de integración.

---

## 🟡 2. In Progress (En Curso)

*(Vacío — Listo para iniciar `ISSUE-001`)*

---

## 🟣 3. In Review / Testing (PR abierto hacia `develop`)

*(Vacío)*

---

## 📋 4. Backlog (Ordenado por Épica y Fase)

### Fase 1 — API & Frontend Web Local (`release/v0.1.0`)
- [ ] **[ISSUE-004](./backlog/USER_STORIES.md#issue-004-endpoints-crud-del-daily-log-y-backlog-appsapi)**: Endpoints CRUD del Daily Log y Backlog (`apps/api`)
- [ ] **[ISSUE-005](./backlog/USER_STORIES.md#issue-005-endpoint-de-detección-y-migración-en-1-clic-de-tareas-pendientes-pasadas-appsapi)**: Endpoint de Detección y Migración en 1 Clic de Tareas Pendientes Pasadas (`apps/api`)
- [ ] **[ISSUE-006](./backlog/USER_STORIES.md#issue-006-endpoint-de-resumen-mensual-para-mini-calendario-y-filtro-por-tags-appsapi)**: Endpoint de Resumen Mensual para Mini-Calendario y Filtro por Tags (`apps/api`)
- [ ] **[ISSUE-007](./backlog/USER_STORIES.md#issue-007-shell-de-aplicación-web-minimalista-react--vite--tailwind--shadcnui--modo-oscuro)**: Shell de Aplicación Web Minimalista (`apps/web`)
- [ ] **[ISSUE-008](./backlog/USER_STORIES.md#issue-008-vista-daily-log-interactiva-captura-rápida-de-bullets-checkboxes-y-tags)**: Vista Daily Log Interactiva (Captura Rápida de Bullets, Checkboxes y Tags) (`apps/web`)
- [ ] **[ISSUE-009](./backlog/USER_STORIES.md#issue-009-ui-de-migración-en-1-clic-bandeja-de-backlog-y-mini-calendario-mensual)**: UI de Migración en 1 Clic, Bandeja de Backlog y Mini-Calendario Mensual (`apps/web`)

### Fase 2 — Despliegue Cloud AWS Always Free (`release/v0.2.0`)
- [ ] **[ISSUE-010](./backlog/USER_STORIES.md#issue-010-pantalla-de-loginsesión-online-y-adaptador-serverless-aws-lambda-appsapi--appsweb)**: Pantalla de Login Online y Adaptador Serverless AWS Lambda
- [ ] **[ISSUE-011](./backlog/USER_STORIES.md#issue-011-infraestructura-como-código-iac--scripts-de-despliegue-aws-always-free-y-cicd)**: Infraestructura AWS Always Free (Lambda + S3/CloudFront + Neon/Supabase Postgres) y Alertas de Presupuesto $0

### Fase 3 — App Mobile Multiplataforma (`release/v0.3.0`)
- [ ] **[ISSUE-012](./backlog/USER_STORIES.md#issue-012-app-mobile-en-react-native-expo-conectada-al-backend-y-a-packagesshared)**: App Mobile en React Native (Expo) consumiendo `packages/shared` y `apps/api`

---

## ✅ 5. Done (Completadas)

- [x] **ISSUE-000**: Ingeniería de Requisitos (`/grill-me`), 6 ADRs, SRS, GitFlow, Reglas en `AGENTS.md`, Hooks de protección (`.env` y ramas GitFlow) y Cuaderno de Desarrollo inicial.
