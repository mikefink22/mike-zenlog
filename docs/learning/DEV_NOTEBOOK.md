# Cuaderno de Desarrollo (Dev Notebook) — Agenda 2026

Este cuaderno es tu espacio de estudio personal dentro del proyecto. Cada vez que introducimos una decisión arquitectónica, librería o patrón nuevo, lo documentamos aquí conectándolo con tecnologías que ya conoces (**Django REST Framework, Angular, Java, React + Vite**).

---

## Índice de Conceptos

1. [Monorepo con `npm workspaces` vs Polirepo](#1-monorepo-con-npm-workspaces-vs-polirepo)
2. [Esquemas Compartidos con Zod (`packages/shared`) vs Serializers de DRF](#2-esquemas-compartidos-con-zod-packagesshared-vs-serializers-de-drf)
3. [Hono Framework vs Express y Django REST Framework](#3-hono-framework-vs-express-y-django-rest-framework)
4. [Drizzle ORM + PGlite vs Django ORM / Hibernate (Java)](#4-drizzle-orm--pglite-vs-django-orm--hibernate-java)
5. [AWS "Always Free" vs "12 Months Free" (Conceptos Cloud)](#5-aws-always-free-vs-12-months-free-conceptos-cloud)
6. [Metodología GitFlow en Equipos de Software](#6-metodología-gitflow-en-equipos-de-software)

---

## 1. Monorepo con `npm workspaces` vs Polirepo

### ¿Qué problema resuelve?
Cuando construyes una plataforma con **Backend API**, **Frontend Web** y **App Mobile**, si creas 3 repositorios separados (*Polirepo*), cada vez que agregas un campo (por ejemplo, `scheduledTime` en una tarea) tienes que:
1. Cambiar el backend.
2. Copiar y pegar la interfaz TypeScript en el repo de la Web.
3. Copiar y pegar la interfaz en el repo Mobile.

Con un **Monorepo (`npm workspaces`)**, todo vive en un solo repositorio organizado en carpetas:
- `packages/shared`: Código común (tipos y validaciones).
- `apps/api`: El backend.
- `apps/web`: El frontend.
- `apps/mobile`: La futura app móvil.

### ¿Por qué `npm workspaces` es seguro y cómo funciona?
En el `package.json` de la raíz declaramos:
```json
{
  "workspaces": ["packages/*", "apps/*"]
}
```
Cuando ejecutas `npm install` en la raíz:
1. `npm` instala todas las dependencias en un único `node_modules` raíz (ahorrando espacio en disco).
2. Crea un enlace simbólico (*symlink*) automático para que `apps/api` y `apps/web` puedan hacer `import { BulletSchema } from '@agenda/shared'` como si fuera una librería instalada de internet, pero leyendo directamente tu código local en tiempo real.

---

## 2. Esquemas Compartidos con Zod (`packages/shared`) vs Serializers de DRF

### ¿Qué problema resuelve?
TypeScript valida los tipos **en tiempo de compilación** (mientras escribes código), pero cuando el código compila a JavaScript y corre en el navegador o recibe un JSON por HTTP en el servidor, **los tipos de TypeScript desaparecen**. Si alguien envía un JSON malicioso o vacío a tu API, una `interface` de TypeScript no lo frena.

**Zod** valida los datos **en tiempo de ejecución (runtime)** y al mismo tiempo **genera el tipo de TypeScript automáticamente**.

### Analogía con lo que ya conoces (DRF y Java)
- En **Django REST Framework (Python)** escribes un `ModelSerializer` o `Serializer` para validar el JSON que entra a un endpoint. Pero luego, en tu frontend (Angular o React), tenías que volver a escribir una `interface` TypeScript y las reglas `Validators.required` del formulario.
- Con **Zod** en `packages/shared`, defines el "Serializer" **una sola vez en TypeScript** y lo usas tanto en el Backend como en el formulario de React:

```typescript
import { z } from 'zod';

// 1. Definimos el esquema de validación (Runtime) - Equivalente a un Serializer de DRF
export const CreateBulletSchema = z.object({
  type: z.enum(['task', 'event', 'note']),
  content: z.string().min(1, 'El contenido no puede estar vacío').max(500),
  scheduledDate: z.string().regex(/^\d{4}-\d{2}-\d{2}$/).nullable(),
});

// 2. Inferimos el tipo estático (Compile-time) automáticamente - Sin duplicar interfaces
export type CreateBulletInput = z.infer<typeof CreateBulletSchema>;
```

---

## 3. Hono Framework vs Express y Django REST Framework

### ¿Qué es Hono?
**Hono** es un framework web moderno para TypeScript/JavaScript construido sobre los **Web Standards** (`Request` y `Response` nativos de `fetch`, los mismos del navegador).

### ¿Por qué lo elegimos sobre Express para ir a AWS Lambda?
- **Express** fue creado en 2010 usando objetos propios de Node (`req`, `res`). Cuando quieres subir Express a **AWS Lambda** (Serverless), necesitas parches que traduzcan el evento de AWS Lambda a un `req/res` falso de Node.
- **Hono** es nativo de ambientes Serverless y Edge:
  - En tu PC local corre con `@hono/node-server` igual que Express.
  - En producción en **AWS Lambda** exportas `handle(app)` desde `hono/aws-lambda` con 0 cambios en tus rutas y un arranque (*cold start*) casi instantáneo.

---

## 4. Drizzle ORM + PGlite vs Django ORM / Hibernate (Java)

### Drizzle ORM: "Si sabes SQL, sabes Drizzle"
A diferencia de ORMs pesados como Hibernate (Java) o Prisma (que levanta un binario en Rust separado), **Drizzle ORM** es una capa ultradelgada en TypeScript donde tus tablas y consultas se ven casi idénticas a SQL real, pero 100% autocompletadas por TypeScript.

### ¿Qué es PGlite (`@electric-sql/pglite`)?
Normalmente, para usar PostgreSQL en desarrollo local tenías dos opciones:
1. Instalar el servicio de PostgreSQL en Windows (que consume RAM en segundo plano).
2. Instalar Docker Desktop (que levanta una máquina virtual WSL2 pesada).

**PGlite** empaqueta el motor **PostgreSQL real oficial compilado a WebAssembly (Wasm)** en un paquete de `npm` de ~3MB:
- En modo desarrollo guarda las tablas en una carpeta `.local-db/` de tu proyecto.
- En modo testing (`npm test`), levanta una base de datos PostgreSQL completa **en memoria RAM en milisegundos** para cada test y la destruye al terminar.
- Cuando despliegas a la nube (Supabase/Neon), Drizzle cambia el conector de `pglite` a `pg` sin tocar ni una sola línea de tus consultas SQL.

---

## 5. AWS "Always Free" vs "12 Months Free" (Conceptos Cloud)

Uno de los errores más comunes al usar la capa gratuita de AWS tras un curso universitario es confundir los dos modelos:

| Servicio AWS | Categoría | Límite Gratuito | ¿Qué pasa al mes 13? |
| :--- | :--- | :--- | :--- |
| **EC2 (`t2.micro` / `t3.micro`)** | 12 Meses Gratis | 750 hs/mes | **Empieza a cobrar** (~$8-$10 USD/mes) |
| **RDS (PostgreSQL/MySQL)** | 12 Meses Gratis | 750 hs/mes + 20 GB | **Empieza a cobrar** (~$15-$20 USD/mes) |
| **AWS Lambda** | **Always Free (Para siempre)** | 1.000.000 requests/mes | **Sigue siendo $0 gratis para siempre** |
| **Amazon CloudFront** | **Always Free (Para siempre)** | 1 TB transferencia + 10M requests/mes | **Sigue siendo $0 gratis para siempre** |
| **Amazon DynamoDB** | **Always Free (Para siempre)** | 25 GB + 25 RCU/WCU (modo aprovisionado) | **Sigue siendo $0 gratis para siempre** |

Por eso nuestra arquitectura usa **AWS Lambda + CloudFront** combinados con **PostgreSQL Serverless (Neon/Supabase)** que sí tiene capa *Always Free* perpetua para SQL.

---

## 6. Metodología GitFlow en Equipos de Software

### Reglas de oro que aplicamos en este repositorio:
1. **`main` es sagrada**: Representa lo que está listo para producción. Cada merge a `main` lleva un tag de versión (`v0.1.0`, `v0.2.0`).
2. **`develop` es el punto de encuentro**: Aquí se integran todas las tareas terminadas.
3. **Nadie trabaja sobre `develop` directo**: Por cada tarjeta del Kanban (`ISSUE-001`, `ISSUE-002`...), creamos una rama `feature/ISSUE-XXX-nombre`.
4. **Definition of Done (DoD)**: Una rama `feature/*` solo puede volver a `develop` cuando:
   - Cumple todos los escenarios Gherkin (`Given / When / Then`).
   - Los tests automatizados (`npm test`) pasan en verde.
   - El código compila sin errores de tipos (`npm run typecheck`).
