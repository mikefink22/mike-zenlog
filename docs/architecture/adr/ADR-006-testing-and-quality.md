# ADR-006: Estrategia de Calidad, Testing (Vitest + Playwright) y Metodología Docs-as-Code

- **Estado**: Aceptado
- **Fecha**: 2026-11-01
- **Contexto**: Se busca trabajar simulando un equipo de ingeniería de software riguroso: trazabilidad desde los requisitos (SRS) -> Historias de Usuario -> Issues -> Criterios de Aceptación (Gherkin) -> Tests automatizados -> Pull Requests bajo GitFlow.

## Decisión

1. **Docs-as-Code en `/docs`**:
   - Todos los requisitos, ADRs, historias de usuario, tablero Kanban (`docs/KANBAN.md`) y el Cuaderno de Desarrollo (`docs/learning/DEV_NOTEBOOK.md`) se versionan en Markdown dentro del repositorio junto con el código.
2. **Pirámide de Testing alineada a Criterios Gherkin**:
   - **Unit & Contract Tests (`packages/shared`)**: **Vitest** para probar esquemas Zod, validaciones de tipos de bullets y reglas de negocio puras.
   - **Integration Tests de API (`apps/api`)**: **Vitest** ejecutando peticiones contra la instancia `app.request()` de Hono respaldada por una base de datos **PGlite en memoria** (aislamiento total y velocidad <1s por suite). También compatible con colecciones de **Postman CLI** para pruebas de contrato HTTP.
   - **Component & E2E Tests (`apps/web`)**: **Vitest + Testing Library** para componentes y **Playwright** para flujos críticos de punta a punta.
3. **CI/CD con GitHub Actions**:
   - Un pipeline en `.github/workflows/ci.yml` que ejecuta `npm run lint`, `npm run typecheck` y `npm test` en cada Pull Request hacia `develop` o `main`.
