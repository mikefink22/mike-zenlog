# ADR-003: Base de Datos Híbrida (PGlite Local + PostgreSQL Serverless) e Infraestructura Cloud AWS Always Free

- **Estado**: Aceptado
- **Fecha**: 2026-11-01
- **Contexto**: El proyecto requiere ejecutarse inicialmente en el entorno local del desarrollador (Windows, sin Docker ni servidor PostgreSQL corriendo como servicio) y luego desplegarse en la nube aprovechando recursos **Gratis para siempre (Always Free)** en AWS sin incurrir en cobros sorpresa tras los 12 meses.

## Hallazgos sobre la Capa Gratuita de AWS ("Always Free" vs "12 Months Free")

- **AWS Lambda + API Gateway**: **Always Free** (1.000.000 de invocaciones/mes y 400.000 GB-s de cómputo por mes gratis para siempre).
- **AWS CloudFront + S3**: **Always Free** en CloudFront (1 TB de transferencia de salida y 10.000.000 de peticiones HTTP/HTTPS al mes gratis para siempre).
- **Amazon RDS (PostgreSQL tradicional)**: **NO es Always Free** (solo gratis por 12 meses; luego pasa a costar ~$15–$20 USD/mes).
- **Alternativas SQL Always Free**:
  - **PostgreSQL Serverless externo (Neon.tech o Supabase)**: Capa gratuita permanente (0.5 GB de almacenamiento PostgreSQL estándar) conectada por `DATABASE_URL` desde AWS Lambda.
  - **Amazon Aurora DSQL**: Motor SQL distribuido serverless de AWS con 1 GB de almacenamiento y 100k DPUs mensuales en Always Free.

## Decisión

1. **En Desarrollo Local y Testing (`NODE_ENV=development | test`)**: Usar **PGlite (`@electric-sql/pglite`)** junto con **Drizzle ORM**. PGlite ejecuta un motor **PostgreSQL real compilado en WebAssembly** que persiste en una carpeta local (`apps/api/.local-db/`) o en memoria para los tests, sin requerir Docker ni servicios de Windows.
2. **En Producción Cloud (`NODE_ENV=production`)**:
   - **Cómputo API**: AWS Lambda + API Gateway HTTP API.
   - **Hosting Web**: AWS S3 + CloudFront.
   - **Base de Datos**: PostgreSQL Serverless Always Free (Supabase / Neon) intercambiable únicamente mediante la variable `DATABASE_URL` en el driver de Drizzle.

## Consecuencias

- **Positivas**: Costo $0 permanente; paridad 100% SQL de PostgreSQL entre local, tests automatizados y producción; cero configuración de servicios pesados en Windows.
- **Negativas / Mitigación**: Los backups de datos locales viven en la carpeta ignorada por Git `.local-db/`, por lo que proveeremos un script de `db:seed` y exportación JSON.
