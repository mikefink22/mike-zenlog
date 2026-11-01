# Historias de Usuario, Issues, Criterios de Aceptación (Gherkin) y Plan de Testing

Este documento desglosa las **Épicas** del proyecto en **Historias de Usuario (HU)** e **Issues accionables (`ISSUE-001` a `ISSUE-012`)**, detallando sus **Criterios de Aceptación en formato Gherkin (`Given / When / Then`)** y qué pruebas automatizadas deben verificarlos antes de cerrar cada PR en GitFlow.

---

## ÉPICA 1: Base del Monorepo, Contratos Compartidos y Persistencia Local (Fase 1 - Core)

### ISSUE-001: Configuración del Monorepo TypeScript y Esquemas Zod Compartidos (`packages/shared`)
- **Historia de Usuario (HU-01)**: Como desarrollador del equipo, quiero un paquete compartido `packages/shared` con los esquemas Zod de `BulletItem`, `User` y filtros de fecha/tags, para validar con una única fuente de verdad tanto en el Backend como en el Frontend y futura App Mobile.
- **Rama GitFlow**: `feature/ISSUE-001-shared-zod-schemas`
- **Criterios de Aceptación (Gherkin)**:
  ```gherkin
  Scenario: Validación exitosa de un Bullet de tipo Tarea
    Given un payload con type="task", content="Estudiar para el parcial #facultad" y scheduledDate="2026-11-01"
    When se valida contra CreateBulletSchema en packages/shared
    Then la validación resulta exitosa
    And el status por defecto se asigna como "pending"
    And el extractor de etiquetas detecta automáticamente el tag "facultad"

  Scenario: Rechazo de un Bullet con contenido vacío o fecha inválida
    Given un payload con content="" o scheduledDate="01-11-2026" (formato incorrecto)
    When se valida contra CreateBulletSchema
    Then la validación falla devolviendo mensajes de error tipados por campo
  ```
- **Plan de Testing**: Tests unitarios con **Vitest** en `packages/shared/src/__tests__/bullet.schema.test.ts`.

---

### ISSUE-002: Capa de Base de Datos con Drizzle ORM y PGlite (`apps/api`)
- **Historia de Usuario (HU-02)**: Como desarrollador, quiero configurar Drizzle ORM con soporte dual (PGlite local/test y PostgreSQL Serverless para producción) con las tablas `users` y `bullets`, para persistir mis datos en SQL real sin depender de Docker en Windows.
- **Rama GitFlow**: `feature/ISSUE-002-drizzle-pglite-db`
- **Criterios de Aceptación (Gherkin)**:
  ```gherkin
  Scenario: Inicialización de esquema SQL y usuario local por defecto
    Given una instancia limpia de PGlite en memoria o carpeta local
    When se ejecutan las migraciones/push de Drizzle y el seed inicial
    Then las tablas "users" y "bullets" existen con sus claves foráneas e índices por fecha
    And existe el usuario local por defecto cuando AUTH_MODE="local_single_user"
  ```
- **Plan de Testing**: Test de integración de base de datos con **Vitest + PGlite in-memory** en `apps/api/src/db/__tests__/db.test.ts`.

---

### ISSUE-003: Autenticación Dual (Modo Local Single-User + Login JWT) (`apps/api`)
- **Historia de Usuario (HU-03)**: Como usuario de la agenda, quiero que en mi PC local la agenda abra directamente en mi sesión sin pedir clave, pero que al activarse el modo online (`AUTH_MODE=jwt`) proteja mis rutas exigiendo un token JWT válido al iniciar sesión.
- **Rama GitFlow**: `feature/ISSUE-003-auth-local-and-jwt`
- **Criterios de Aceptación (Gherkin)**:
  ```gherkin
  Scenario: Acceso directo en modo local de desarrollo
    Given el servidor API configurado con NODE_ENV="development" y AUTH_MODE="local_single_user"
    When realizo GET /api/auth/me sin enviar cabeceras de autorización
    Then recibo HTTP 200 con los datos del usuario local por defecto

  Scenario: Protección de rutas y Login en modo JWT
    Given el servidor API configurado con AUTH_MODE="jwt"
    When realizo GET /api/bullets sin token JWT
    Then recibo HTTP 401 Unauthorized
    When envío POST /api/auth/login con credenciales válidas
    Then recibo HTTP 200 con un token JWT válido que me permite acceder a GET /api/bullets
  ```
- **Plan de Testing**: Tests de integración con **Vitest** sobre Hono (`app.request`) probando ambos modos de autenticación.

---

## ÉPICA 2: API REST/RPC del Bullet Journal (`apps/api`)

### ISSUE-004: Endpoints CRUD del Daily Log y Backlog (`apps/api`)
- **Historia de Usuario (HU-04)**: Como usuario, quiero poder crear, listar por fecha (o sin fecha para el Backlog), editar, completar y eliminar bullets desde la API, para gestionar las entradas de mi día a día.
- **Rama GitFlow**: `feature/ISSUE-004-bullets-crud-api`
- **Criterios de Aceptación (Gherkin)**:
  ```gherkin
  Scenario: Crear y listar bullets del día actual
    Given un usuario autenticado en la fecha "2026-11-01"
    When envía POST /api/bullets con type="task", content="Comprar repuestos #personal", scheduledDate="2026-11-01"
    And envía POST /api/bullets con type="event", content="Entrega TP AWS #facultad", scheduledDate="2026-11-01", scheduledTime="18:00"
    And consulta GET /api/bullets?date=2026-11-01
    Then recibe HTTP 200 con los 2 bullets ordenados por posición/creación y sus tags extraídos

  Scenario: Completar una tarea y filtrar por Backlog
    Given una tarea existente con status="pending"
    When envía PATCH /api/bullets/:id con status="completed"
    Then el estado persiste como "completed"
    When consulta GET /api/bullets?backlog=true
    Then recibe únicamente los bullets cuyo scheduledDate es null
  ```
- **Plan de Testing**: Tests de integración HTTP en **Vitest** + colección exportable para **Postman CLI**.

---

### ISSUE-005: Endpoint de Detección y Migración en 1 Clic de Tareas Pendientes Pasadas (`apps/api`)
- **Historia de Usuario (HU-05)**: Como usuario, quiero saber cuántas tareas quedaron sin completar en días anteriores a hoy y poder migrarlas a hoy, enviarlas al backlog o descartarlas con una sola acción, para mantener mi agenda limpia al estilo Bullet Journal.
- **Rama GitFlow**: `feature/ISSUE-005-task-migration-api`
- **Criterios de Aceptación (Gherkin)**:
  ```gherkin
  Scenario: Detectar tareas vencidas no completadas
    Given una tarea con status="pending" en "2026-10-30" y una tarea con status="completed" en "2026-10-30"
    When consulto GET /api/bullets/overdue?beforeDate=2026-11-01
    Then recibo únicamente la tarea con status="pending" del "2026-10-30"

  Scenario: Migrar tareas pendientes al día de hoy en 1 clic
    Given una tarea pendiente en "2026-10-30" con id="task-1"
    When envío POST /api/bullets/migrate con action="move_to_date", targetDate="2026-11-01" e ids=["task-1"]
    Then la tarea aparece asignada al día "2026-11-01" con estado "pending" y registro de migración
  ```
- **Plan de Testing**: Tests de integración en **Vitest** cubriendo las 3 acciones (`move_to_date`, `move_to_backlog`, `discard`).

---

### ISSUE-006: Endpoint de Resumen Mensual para Mini-Calendario y Filtro por Tags (`apps/api`)
- **Historia de Usuario (HU-06)**: Como usuario, quiero que el mini-calendario mensual muestre qué días del mes tienen actividad (tareas pendientes, completadas o eventos) y poder filtrar por `#etiqueta`, para encontrar rápido mis compromisos.
- **Rama GitFlow**: `feature/ISSUE-006-calendar-summary-and-tags-api`
- **Criterios de Aceptación (Gherkin)**:
  ```gherkin
  Scenario: Obtener resumen de actividad de un mes (YYYY-MM)
    Given bullets cargados en los días "2026-11-01" y "2026-11-15"
    When consulto GET /api/bullets/calendar-summary?month=2026-11
    Then recibo un mapa por día con el conteo de tareas pendientes, tareas completadas y eventos de ese mes
  ```
- **Plan de Testing**: Tests de integración en **Vitest**.

---

## ÉPICA 3: Interfaz Web Minimalista y Responsive (`apps/web`)

### ISSUE-007: Shell de Aplicación Web Minimalista (React + Vite + Tailwind + shadcn/ui + Modo Oscuro)
- **Historia de Usuario (HU-07)**: Como usuario, quiero una interfaz limpia, sin ruido visual, responsive para escritorio y móvil, con soporte de tema claro/oscuro y navegación rápida entre "Hoy", "Backlog" y "Mini-Calendario".
- **Rama GitFlow**: `feature/ISSUE-007-web-minimalist-shell`
- **Criterios de Aceptación (Gherkin)**:
  ```gherkin
  Scenario: Renderizado responsive en Desktop y Mobile
    Given el usuario abre la aplicación web
    Then en escritorio visualiza el panel lateral (Mini-Calendario, Backlog y Tags) junto al Daily Log central
    And en pantallas móviles (<768px) visualiza el Daily Log a pantalla completa con navegación inferior/drawer táctil
  ```
- **Plan de Testing**: Tests de componentes con **Vitest + React Testing Library**.

---

### ISSUE-008: Vista Daily Log Interactiva (Captura Rápida de Bullets, Checkboxes y Tags)
- **Historia de Usuario (HU-08)**: Como usuario, quiero escribir en una barra de captura rápida en mi vista del día (pudiendo elegir Tarea `•`, Evento `○` o Nota `—`), marcar tareas con un clic y ver las etiquetas resaltadas, para organizar mi día en segundos.
- **Rama GitFlow**: `feature/ISSUE-008-web-daily-log-view`
- **Criterios de Aceptación (Gherkin)**:
  ```gherkin
  Scenario: Creación rápida por teclado y marcado de tarea
    Given el usuario está en la vista del día "Hoy"
    When escribe "Repasar ADR de AWS #estudio" y presiona Enter
    Then el nuevo bullet aparece inmediatamente en la lista del día con el chip "#estudio"
    When hace clic en el checkbox de la tarea
    Then la tarea se muestra completada (tachada sutilmente) y sincronizada con la API
  ```
- **Plan de Testing**: Tests de integración UI con **Vitest** y test E2E con **Playwright**.

---

### ISSUE-009: UI de Migración en 1 Clic, Bandeja de Backlog y Mini-Calendario Mensual
- **Historia de Usuario (HU-09)**: Como usuario, quiero ver el aviso de tareas pendientes de días pasados para migrarlas a hoy con 1 clic, gestionar mi Backlog sin fecha y saltar a cualquier día desde el Mini-Calendario.
- **Rama GitFlow**: `feature/ISSUE-009-web-migration-backlog-calendar`
- **Criterios de Aceptación (Gherkin)**:
  ```gherkin
  Scenario: Banner de migración consciente al iniciar el día
    Given existen 2 tareas pendientes de días anteriores
    When el usuario abre la vista de "Hoy"
    Then visualiza el banner "Tienes 2 tareas pendientes de días anteriores"
    When presiona "Traer todas a Hoy"
    Then las 2 tareas pasan a la lista de "Hoy" y el banner desaparece
  ```
- **Plan de Testing**: Tests de componentes y flujo E2E completo con **Playwright** (cerrando la Fase 1 -> `release/v0.1.0`).

---

## ÉPICA 4: Despliegue Cloud Zero-Cost en AWS y Sesión Online (Fase 2)

### ISSUE-010: Pantalla de Login/Sesión Online y Adaptador Serverless AWS Lambda (`apps/api` + `apps/web`)
- **Historia de Usuario (HU-10)**: Como usuario, quiero iniciar sesión en mi agenda cuando esté desplegada en internet y que mi API funcione dentro de AWS Lambda conectada a PostgreSQL Serverless Always Free (Neon/Supabase).
- **Rama GitFlow**: `feature/ISSUE-010-cloud-auth-and-lambda-adapter`

### ISSUE-011: Infraestructura como Código (IaC / Scripts de Despliegue AWS Always Free) y CI/CD
- **Historia de Usuario (HU-11)**: Como ingeniero cloud, quiero desplegar el Backend en AWS Lambda + API Gateway y el Frontend en S3 + CloudFront con alertas de presupuesto ($0 budget alert), asegurando costo $0 perpetuo.
- **Rama GitFlow**: `feature/ISSUE-011-aws-always-free-deploy`

---

## ÉPICA 5: Aplicación Mobile Multiplataforma (Fase 3)

### ISSUE-012: App Mobile en React Native (Expo) conectada al Backend y a `packages/shared`
- **Historia de Usuario (HU-12)**: Como usuario móvil, quiero abrir mi agenda desde una app nativa en mi celular que consuma la misma API y esquemas Zod de `packages/shared`.
- **Rama GitFlow**: `feature/ISSUE-012-mobile-expo-app`
