# Especificación de Requisitos de Software (SRS) — Agenda 2026 (Bullet Journal Minimalista)

- **Versión**: 1.0.0
- **Fecha**: 2026-11-01
- **Autor**: Equipo de Ingeniería (Usuario + Antigravity)

---

## 1. Propósito y Alcance del Producto

Crear una agenda personal digital inspirada en el método **Bullet Journal (BuJo)**, con un diseño **minimalista y libre de fricción** para evitar la sobrecarga de funcionalidades.
El sistema sigue una hoja de ruta en 3 fases evolutivas sobre la misma base de código:
- **Fase 1 (MVP Local)**: Uso inmediato en máquina local (Windows) con persistencia SQL embebida (PGlite).
- **Fase 2 (Despliegue Web Responsive & Cloud Zero-Cost)**: Despliegue en AWS (Lambda + S3/CloudFront) con PostgreSQL Serverless Always Free y login seguro para acceso desde navegador en escritorio y celular.
- **Fase 3 (App Mobile)**: Aplicación móvil en React Native (Expo) consumiendo la misma API y esquemas compartidos (`packages/shared`).

---

## 2. Modelo de Dominio (Conceptos Centrales)

### 2.1 El "Bullet" (Unidad Atómica de Información)
En lugar de tener múltiples módulos desconectados, toda entrada de la agenda es un **Bullet** (`BulletItem`):

| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| `id` | `UUID` | Identificador único del bullet |
| `userId` | `UUID` | Usuario propietario del ítem |
| `type` | `'task' \| 'event' \| 'note'` | Tipo de entrada Bullet Journal: Tarea (`•`), Evento (`○`), Nota (`—`) |
| `content` | `string` (1..500) | Texto descriptivo del bullet (soporta etiquetas inline como `#facultad`) |
| `status` | `'pending' \| 'completed' \| 'migrated' \| 'discarded'` | Aplicable a `task` (para `event` y `note` se mantiene en `pending` o `completed`) |
| `scheduledDate` | `YYYY-MM-DD \| null` | Fecha del Daily Log al que pertenece. Si es `null`, vive en el **Backlog (Bandeja de Entrada)** |
| `scheduledTime` | `HH:mm \| null` | Hora opcional (principalmente para `event`, ej. `14:30`) |
| `tags` | `string[]` | Etiquetas extraídas o asignadas (ej. `["facultad", "personal"]`) |
| `migratedFromId` | `UUID \| null` | Trazabilidad si este bullet fue migrado desde un día anterior |
| `position` | `integer` | Orden del ítem dentro de su lista del día o backlog |
| `createdAt` / `updatedAt` | `ISO Timestamp` | Fechas de auditoría |

---

## 3. Requisitos Funcionales (RF)

- **RF-01 (Daily Log Unificado)**: El usuario debe poder ver la fecha seleccionada (por defecto "Hoy") y agregar rápidamente un bullet eligiendo su tipo (`task`, `event`, `note`), hora opcional y etiquetas (`#tag`).
- **RF-02 (Gestión de Estado de Tareas)**: El usuario debe poder marcar/desmarcar una tarea como completada (`completed` / `pending`), editar su texto o eliminarla/descartarla (`discarded`).
- **RF-03 (Migración Consciente en 1 Clic)**: Cuando existan tareas con estado `pending` en fechas anteriores a hoy (`scheduledDate < today`), el sistema debe mostrar un banner no intrusivo indicando la cantidad de tareas pendientes pasadas y permitiendo revisar y ejecutar con 1 clic:
  - **Migrar a Hoy**: Actualiza la tarea a hoy (o marca la original como `migrated` y la mueve a hoy).
  - **Enviar al Backlog**: Quita la fecha (`scheduledDate = null`) para hacerla más adelante.
  - **Descartar**: La marca como `discarded` si ya no es relevante.
- **RF-04 (Bandeja de Entrada / Backlog)**: El usuario debe contar con una lista de ítems sin fecha asignada (`scheduledDate = null`) y poder asignarle fecha ("Pasar a Hoy" o a un día específico) con un clic.
- **RF-05 (Mini-Calendario Mensual con Indicadores)**: El usuario debe poder navegar entre días mediante un calendario mensual compacto que muestre puntos/indicadores en los días que tienen eventos o tareas pendientes/completadas.
- **RF-06 (Filtrado por Etiquetas `#tags`)**: El usuario debe poder filtrar su vista actual o buscar bullets por etiqueta (ej. `#facultad`, `#trabajo`, `#salud`).
- **RF-07 (Autenticación Dual Local/Cloud)**: En entorno local puede operar con sesión automática local; en producción exige login con email y contraseña mediante JWT.

---

## 4. Requisitos No Funcionales (RNF)

- **RNF-01 (Minimalismo y Velocidad de Captura)**: Agregar un nuevo bullet desde que se abre la app no debe requerir más de 1 clic o pulsar `Enter` en el input principal (soporte de prefijos rápidos opcionales: `t ` para tarea, `e ` para evento, `n ` para nota).
- **RNF-02 (Responsive Mobile-First)**: La interfaz web debe adaptarse perfectamente desde pantallas de `360px` (móvil vertical) hasta monitores de escritorio (`1440px+`).
- **RNF-03 (Costo Operativo $0 en Cloud)**: Toda la arquitectura desplegada en la Fase 2 debe mantenerse estrictamente dentro de los límites **Always Free** de AWS y PostgreSQL Serverless.
- **RNF-04 (Seguridad de Secretos)**: Ninguna credencial o archivo `.env` puede ser versionado ni leído por herramientas automatizadas de IA (solo `.env.example`).
- **RNF-05 (Cobertura de Criterios de Aceptación)**: Cada Historia de Usuario debe contar con tests automatizados que verifiquen sus escenarios Gherkin antes de fusionarse a `develop`.
