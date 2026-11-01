# Guía de Contribución (CONTRIBUTING.md)

¡Bienvenido al repositorio de **Agenda 2026**!

## Flujo de Trabajo Resumido (Ciclo de Vida de un Issue)

1. **Consultar el Tablero Kanban**: Abre [docs/KANBAN.md](./docs/KANBAN.md) y elige el siguiente Issue de la columna `Ready`.
2. **Revisar Requisitos y Criterios Gherkin**: Lee el detalle del Issue en [docs/backlog/USER_STORIES.md](./docs/backlog/USER_STORIES.md).
3. **Crear Rama Feature desde `develop`**:
   ```powershell
   git checkout develop
   git checkout -b feature/ISSUE-XXX-nombre-corto
   ```
4. **Implementar + Escribir Tests**:
   - Desarrollar el código respetando los ADRs en `docs/architecture/adr/`.
   - Escribir los tests en Vitest/Playwright que validan cada criterio `Given / When / Then`.
   - Si hay variables de entorno nuevas, documentarlas **solo** en `.env.example`.
5. **Actualizar el Cuaderno de Desarrollo**:
   - Documentar los conceptos nuevos aprendidos en [docs/learning/DEV_NOTEBOOK.md](./docs/learning/DEV_NOTEBOOK.md).
6. **Ejecutar Verificaciones**:
   ```powershell
   npm run typecheck
   npm test
   ```
7. **Pull Request hacia `develop`**:
   - Actualizar el estado del Issue en `docs/KANBAN.md`.
   - Hacer commit siguiendo Conventional Commits y abrir PR hacia `develop`.
